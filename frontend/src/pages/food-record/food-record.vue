<template>
  <view class="container">
    <view class="record-list">
      <view class="record-item" v-for="(record, index) in records" :key="index">
        <view class="record-header">
          <text class="food-name">{{ record.foodName }}</text>
          <text class="record-time">{{ record.recordTime }}</text>
        </view>
        <view class="record-details">
          <view class="detail-item">
            <text class="label">重量</text>
            <text class="value">{{ record.weight }}克</text>
          </view>
          <view class="detail-item">
            <text class="label">每100克热量</text>
            <text class="value">{{ record.caloriesPer100g }}卡</text>
          </view>
          <view class="detail-item">
            <text class="label">总热量</text>
            <text class="value total">{{ record.totalCalories }}卡</text>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  data() {
    return {
      records: [
        // {
        //   foodName: "苹果",
        //   caloriesPer100g: 52,
        //   weight: 200,
        //   recordTime: "2024-03-20 12:30",
        //   totalCalories: 104
        // },
        // {
        //   foodName: "米饭",
        //   caloriesPer100g: 116,
        //   weight: 150,
        //   recordTime: "2024-03-20 18:15",
        //   totalCalories: 174
        // },
        // {
        //   foodName: "鸡胸肉",
        //   caloriesPer100g: 165,
        //   weight: 100,
        //   recordTime: "2024-03-20 18:15",
        //   totalCalories: 165
        // },
        // {
        //   foodName: "西兰花",
        //   caloriesPer100g: 34,
        //   weight: 200,
        //   recordTime: "2024-03-20 18:15",
        //   totalCalories: 68
        // },
        // {
        //   foodName: "牛奶",
        //   caloriesPer100g: 54,
        //   weight: 250,
        //   recordTime: "2024-03-20 08:00",
        //   totalCalories: 135
        // }
      ]
    }
  },
  onLoad() {
    this.fetchRecords()
  },
  methods: {
    async fetchRecords() {
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
          this.records = response.data
        }
      } catch (err) {
        console.error('获取记录失败：', err)
        uni.showToast({
          title: '获取记录失败',
          icon: 'none'
        })
      }
    }
  }
}
</script>

<style scoped>
.container {
  padding: 20rpx;
  min-height: 100vh;
  background-color: #f8f8f8;
}

.record-list {
  padding: 20rpx;
}

.record-item {
  background-color: #ffffff;
  border-radius: 12rpx;
  padding: 30rpx;
  margin-bottom: 20rpx;
  box-shadow: 0rpx 2rpx 8rpx 2rpx #cccccc;
}

.record-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
  padding-bottom: 20rpx;
  border-bottom: 1rpx solid #eeeeee;
}

.food-name {
  font-size: 34rpx;
  font-weight: 500;
  color: #333333;
}

.record-time {
  font-size: 28rpx;
  color: #999999;
}

.record-details {
  display: flex;
  justify-content: space-between;
}

.detail-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.label {
  font-size: 24rpx;
  color: #666666;
  margin-bottom: 8rpx;
}

.value {
  font-size: 28rpx;
  color: #333333;
  font-weight: 500;
}

.value.total {
  color: #4cd964;
  font-weight: bold;
}
</style> 