<template>
  <view class="weight-progress-section">
    <view class="progress-wrapper">
      <view class="progress-container">
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
        const weightResponse = await wx.cloud.callContainer({
          path: '/api/weight/goal/getGoal',
          method: 'GET',
          header: {
            'X-WX-SERVICE': 'springboot-glwv', // ⚠️ 替换为实际服务名
            'Authorization': 'Bearer ' + uni.getStorageSync('token'),
            'Content-Type': 'application/json'
          }
        })

        const data = weightResponse.data
        this.initialWeight = data.startWeight
        this.currentWeight = data.currentWeight
        this.targetWeight = data.targetWeight
        this.weightData = [
          { label: '初始体重', value: this.initialWeight },
          { label: '当前体重', value: this.currentWeight },
          { label: '目标体重', value: this.targetWeight }
        ]

      } catch (err) {
        console.error('获取体重数据失败：', err)
      }
    },
  },
  mounted() {
    this.getWeightData()
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
  min-height: 150rpx;
}

.progress-container {
  position: relative;
  width: auto;
  padding-top: 0;
  text-align: center;
}

.progress-info {
  position: static;
  transform: none;
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
  margin-top: 40rpx;
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