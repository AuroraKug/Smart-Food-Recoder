<template>
  <view class="container">
    <SearchBar />
    <view class="food-list-container">
      <view class="food-list-title">食物列表</view>
      <view class="food-item" v-for="(food, index) in foodList" :key="index">
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
import SearchBar from '@/components/SearchBar.vue'
export default {
  components: {
    SearchBar
  },
  data() {
    return {
      foodList: [
        {
          name: '黑咖啡',
          calories: 1,
          image: '/static/logo.png' // 替换为实际的咖啡图片路径
        },
        {
          name: '苹果',
          calories: 52,
          image: '/static/logo.png' // 替换为实际的食物图片路径
        },
        {
          name: '香蕉',
          calories: 90,
          image: '/static/logo.png' // 替换为实际的食物图片路径
        },
        {
          name: '鸡蛋',
          calories: 68,
          image: '/static/logo.png' // 替换为实际的食物图片路径
        },
        {
          name: '面包',
          calories: 265,
          image: '/static/logo.png' // 替换为实际的食物图片路径
        }
      ]
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
  box-shadow: 0rpx 4rpx 8rpx 2rpx #cccccc;
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