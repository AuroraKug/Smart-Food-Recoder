<template>
  <view class="weight-progress-section">
    <view class="progress-wrapper">
      <view class="progress-container">
        <canvas canvas-id="progressCanvas" class="progress-canvas"></canvas>
        <view class="progress-info">
          <view class="progress-value" v-if="!isGoalAchieved">
            {{ weightChange }}<text class="unit">kg</text>
          </view>
          <view class="progress-value completion-text" v-else>
            完成目标
          </view>
          <text class="progress-label">{{ progressLabel }}</text>
        </view>
      </view>
    </view>

    <view class="weight-data">
      <view class="data-item" v-for="(item, index) in weightData" :key="index">
        <text class="data-label">{{ item.label }}</text>
        <text class="data-value">{{ item.value }}<text class="unit">kg</text></text>
      </view>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'
export default {
  props: {
    isCanvasVisible: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      initialWeight: 68.5,
      currentWeight: 64.2,
      targetWeight: 60.0,
      weightData: [
        { label: '初始体重', value: 68.5 },
        { label: '当前体重', value: 64.2 },
        { label: '目标体重', value: 60.0 }
      ]
    }
  },
  computed: {
    weightChange() {
      return Math.abs(this.initialWeight - this.currentWeight).toFixed(1)
    },
    progressPercentage() {
      const total = Math.abs(this.initialWeight - this.targetWeight)
      const current = Math.abs(this.initialWeight - this.currentWeight)
      
      // 判断是增重还是减重目标
      const isWeightGain = this.targetWeight > this.initialWeight
      
      if (isWeightGain) {
        // 增重目标
        if (this.currentWeight < this.initialWeight) {
          return 0 // 如果当前体重小于初始体重，进度为0
        }
        return Math.min(100, Math.max(0, (current / total) * 100))
      } else {
        // 减重目标
        if (this.currentWeight > this.initialWeight) {
          return 0 // 如果当前体重大于初始体重，进度为0
        }
        return Math.min(100, Math.max(0, (current / total) * 100))
      }
    },
    isGoalAchieved() {
      if (this.targetWeight > this.initialWeight) {
        return this.currentWeight >= this.targetWeight
      } else {
        return this.currentWeight <= this.targetWeight
      }
    },
    progressLabel() {
      if (this.isGoalAchieved) {
        return '目标已完成'
      }
      const isWeightGain = this.targetWeight > this.initialWeight
      if (isWeightGain) {
        return this.currentWeight < this.initialWeight ? '需要增重' : '已增体重'
      } else {
        return this.currentWeight > this.initialWeight ? '需要减重' : '已减体重'
      }
    }
  },
  methods: {
    async getWeightData() {
      try {
        const weightResponse = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/weight/goal/getGoal',
            method: 'GET',
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })
        this.initialWeight = weightResponse.data.startWeight
        this.currentWeight = weightResponse.data.currentWeight
        this.targetWeight = weightResponse.data.targetWeight
        this.weightData = [
          { label: '初始体重', value: this.initialWeight },
          { label: '当前体重', value: this.currentWeight },
          { label: '目标体重', value: this.targetWeight }
        ]
      } catch (err) {
        console.error(err)
      }
    },
    drawProgressCircle() {
      const query = uni.createSelectorQuery().in(this)
      query.select('.progress-container').boundingClientRect((rect) => {
        if (!rect) return

        const width = rect.width
        const height = rect.height
        const centerX = width / 2
        const centerY = height / 2
        const radius = Math.min(width, height) / 3
        const lineWidth = Math.min(12, radius / 10)
        const ctx = uni.createCanvasContext('progressCanvas', this)
        ctx.clearRect(0, 0, width, height)

        // 绘制背景圆
        ctx.beginPath()
        ctx.arc(centerX, centerY, radius, 0, Math.PI * 2, false)
        ctx.setStrokeStyle('#f0f0f0')
        ctx.setLineWidth(lineWidth)
        ctx.setLineCap('round')
        ctx.stroke()

        // 绘制进度圆
        const endAngle = (Math.PI * 2 * this.progressPercentage / 100)
        ctx.beginPath()
        ctx.arc(centerX, centerY, radius, -Math.PI / 2, -Math.PI / 2 + endAngle, false)
        ctx.setStrokeStyle(this.isGoalAchieved ? '#4cd964' : '#4cd964')
        ctx.setLineWidth(lineWidth)
        ctx.setLineCap('round')
        ctx.stroke()

        // 绘制进度点
        const dotX = centerX + radius * Math.cos(-Math.PI / 2 + endAngle)
        const dotY = centerY + radius * Math.sin(-Math.PI / 2 + endAngle)
        ctx.beginPath()
        ctx.arc(dotX, dotY, lineWidth / 2, 0, Math.PI * 2)
        ctx.setFillStyle(this.isGoalAchieved ? '#4cd964' : '#4cd964')
        ctx.fill()

        ctx.draw()
      }).exec()
    }
  },
  mounted() {
    this.getWeightData()
    this.drawProgressCircle()
    uni.getSystemInfo({
      success: (res) => {
        this.windowWidth = res.windowWidth
      }
    })
    uni.onWindowResize(() => {
      this.drawProgressCircle()
    })
    // 监听体重更新事件
    uni.$on('weight-updated', () => {
      this.getWeightData()
    })
    // 监听体重数据刷新事件
    uni.$on('refresh-weight-data', () => {
      this.getWeightData()
    })
  },
  beforeDestroy() {
    // 移除事件监听
    uni.$off('weight-updated')
    uni.$off('refresh-weight-data')
  },
  watch: {
    currentWeight() {
      this.drawProgressCircle()
    }
  }
}
</script>

<style scoped>
.weight-progress-section {
  background: white;
  border-radius: 16rpx;
  padding: 30rpx;
  margin: 12rpx auto 24rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
  width: 90%;
}

.progress-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 260rpx;
}

.progress-container {
  position: relative;
  width: 100%;
  padding-top: 43.33%;
  background: transparent;
}

.progress-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100% !important;
  height: 100% !important;
  display: block;
}

.progress-info {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.progress-value {
  font-size: 56rpx;
  font-weight: bold;
  color: #4cd964;
  line-height: 1.2;
}

.completion-text {
  font-size: 36rpx;
}

.progress-label {
  font-size: 24rpx;
  color: #999;
  display: block;
}

.weight-data {
  display: flex;
  justify-content: space-around;
  margin-top: 20rpx;
}

.data-item {
  text-align: center;
  flex: 1;
}

.data-label {
  font-size: 24rpx;
  color: #999;
  display: block;
  margin-bottom: 8rpx;
}

.data-value {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.unit {
  font-size: 24rpx;
  color: #999;
  margin-left: 4rpx;
}
</style>