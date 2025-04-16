<template>
  <view class="health-card">
    <text class="title">今日健康数据</text>
    <view class="data-row">
      <view class="data-item">
        <text>摄入</text>
        <text class="cal-value">{{ totalCalories }}</text>
        <text>kcal</text>
      </view>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  data() {
    return {
      totalCalories: 0
    }
  },
  mounted() {
    this.fetchTodayCalories()
    // 监听健康数据刷新事件
    uni.$on('refresh-health-data', () => {
      this.fetchTodayCalories()
    })
    // 监听食物记录事件
    uni.$on('food-recorded', () => {
      this.fetchTodayCalories()
    })
  },
  beforeDestroy() {
    // 移除事件监听
    uni.$off('refresh-health-data')
    uni.$off('food-recorded')
  },
  methods: {
    async fetchTodayCalories() {
      try {
        const res = await wx.cloud.callContainer({
          path: '/api/food/record/today',
          method: 'GET',
          header: {
            'X-WX-SERVICE': 'springboot-glwv', // ⚠️ 替换为你的实际云托管服务名
            'Authorization': 'Bearer ' + uni.getStorageSync('token'),
            'Content-Type': 'application/json'
          }
        })
        if (res.data && res.data.totalCalories !== undefined) {
          this.totalCalories = res.data.totalCalories.toLocaleString()
        } else {
          this.totalCalories = '0'
        }

      } catch (err) {
        console.error('获取今日卡路里失败：', err)
        this.totalCalories = '0'
      }
    }

  }
}
</script>

<style scoped>
.health-card {
  background: white;
  border-radius: 16rpx;
  padding: 24rpx;
  margin: 0 auto 24rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
  width: 90%;
}

.title {
  font-size: 30rpx;
  font-weight: bold;
  margin-bottom: 20rpx;
  display: block;
}

.data-row {
  display: flex;
  justify-content: space-around;
}

.data-item {
  text-align: center;
}

.cal-value {
  font-size: 50rpx;
  font-weight: bold;
  display: block;
  color: #000;
}
</style>