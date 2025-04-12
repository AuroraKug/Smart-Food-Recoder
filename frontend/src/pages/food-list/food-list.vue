<template>
  <view class="container">
    <view class="food-list-container">
      <view class="food-list-title">{{ searchKeyword ? '搜索结果' : '食物列表' }}</view>
      <view class="food-item" v-for="(food, index) in foodList" :key="index" @click="selectFood(food)">
        <image :src="food.image" class="food-image" mode="widthFix"></image>
        <view class="food-info">
          <text class="food-name">{{ food.name }}</text>
          <text class="food-calories">{{ food.calories }}卡/100克</text>
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
      foodList: [],
      searchKeyword: ''
    }
  },
  onLoad(options) {
    // 获取URL参数中的搜索关键词
    if (options.keyword) {
      this.searchKeyword = decodeURIComponent(options.keyword)
      this.handleSearch(this.searchKeyword)
    }
  },
  methods: {
    async handleSearch(keyword) {
      try {
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/food/search',
            method: 'GET',
            data: {
              keyword: keyword
            },
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

        if (response.statusCode === 200) {
          this.foodList = response.data
        }
      } catch (err) {
        console.error('搜索失败：', err)
        uni.showToast({
          title: '搜索失败',
          icon: 'none'
        })
      }
    },
    selectFood(food) {
      // 触发选择食物事件，供父组件使用
      this.$emit('select', food)
    }
  }
}
</script>

<style scoped>
.container {
  padding: 20rpx;
}

.food-list-container {
  padding: 20rpx;
  min-height: 100vh;
}

.food-list-title {
  font-size: 40rpx;
  font-weight: bold;
  color: #333333;
  text-align: center;
  margin-bottom: 30rpx;
}

.food-item {
  display: flex;
  align-items: center;
  padding: 20rpx;
  background-color: #ffffff;
  border-radius: 12rpx;
  margin: 0 auto 20rpx;
  box-shadow: 0rpx 2rpx 8rpx 2rpx #cccccc;
  overflow: hidden;
}

.food-image {
  width: 120rpx;
  height: 120rpx;
  border-radius: 12rpx;
  margin-right: 24rpx;
  background-color: #f0f0f0;
}

.food-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.food-name {
  font-size: 34rpx;
  color: #333333;
  font-weight: 500;
  line-height: 1.2;
  margin-bottom: 8rpx;
}

.food-calories {
  font-size: 28rpx;
  color: #666666;
  line-height: 1.2;
}
</style>