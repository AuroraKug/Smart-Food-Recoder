<template>
  <view class="recent-records">
    <view class="section-title">
      <text>最近记录</text>
      <text class="more" @click="navigateToRecords">查看更多 ></text>
    </view>
    <view class="record-list">
      <view class="record-item" v-for="(record, index) in recentRecords" :key="index">
        <text>{{ record.foodName }}</text>
        <text>{{ record.totalCalories }} kcal</text>
      </view>
      <view v-if="recentRecords.length === 0" class="empty-tip">
        <text>暂无记录</text>
      </view>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  data() {
    return {
      recentRecords: []
    }
  },
  created() {
    this.fetchRecentRecords()
  },
  methods: {
    async fetchRecentRecords() {
      try {
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/food/record/user',
            method: 'GET',
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

        if (response.statusCode === 200) {
          // 只取最近三条记录
          this.recentRecords = response.data.slice(0, 3)
        }
      } catch (err) {
        console.error('获取记录失败：', err)
        uni.showToast({
          title: '获取记录失败',
          icon: 'none'
        })
      }
    },
    navigateToRecords() {
      uni.navigateTo({
        url: '/pages/food-record/food-record'
      })
    }
  }
}
</script>

<style scoped>
.recent-records {
  background: white;
  border-radius: 16rpx;
  padding: 24rpx;
  width: 90%;
  margin: 0 auto;
}

.section-title {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20rpx;
}

.more {
  color: #888;
}

.record-item {
  display: flex;
  justify-content: space-between;
  padding: 16rpx 0;
  border-bottom: 1rpx solid #eee;
}

.empty-tip {
  text-align: center;
  padding: 20rpx 0;
  color: #999;
}
</style>