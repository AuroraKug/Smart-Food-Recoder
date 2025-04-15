<template>
  <view class="weight-progress-section">
    <view class="progress-wrapper">
      <view class="progress-container">
        <canvas canvas-id="progressCanvas" class="progress-canvas"></canvas>
        <view class="progress-info">
          <view class="progress-value">{{ weightLost }}<text class="unit">kg</text></view>
          <text class="progress-label">已减体重</text>
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
    weightLost() {
      return (this.initialWeight - this.currentWeight).toFixed(1)
    },
    progressPercentage() {
      const total = this.initialWeight - this.targetWeight
      const lost = this.initialWeight - this.currentWeight
      return Math.min(100, Math.max(0, (lost / total) * 100))
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
        console.error(error)
      }
      console.log('weightData', this.weightData)
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

        ctx.beginPath()
        ctx.arc(centerX, centerY, radius, 0, Math.PI * 2, false)
        ctx.setStrokeStyle('#f0f0f0')
        ctx.setLineWidth(lineWidth)
        ctx.setLineCap('round')
        ctx.stroke()

        const endAngle = (Math.PI * 2 * this.progressPercentage / 100)
        ctx.beginPath()
        ctx.arc(centerX, centerY, radius, -Math.PI / 2, -Math.PI / 2 + endAngle, false)
        ctx.setStrokeStyle('#4cd964')
        ctx.setLineWidth(lineWidth)
        ctx.setLineCap('round')
        ctx.stroke()

        const dotX = centerX + radius * Math.cos(-Math.PI / 2 + endAngle)
        const dotY = centerY + radius * Math.sin(-Math.PI / 2 + endAngle)
        ctx.beginPath()
        ctx.arc(dotX, dotY, lineWidth / 2, 0, Math.PI * 2)
        ctx.setFillStyle('#4cd964')
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