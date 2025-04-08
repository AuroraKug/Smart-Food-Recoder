<template>
  <view class="health-card">
    <text class="title">今日健康数据</text>
    <view class="data-row">
      <view class="data-item">
        <text>摄入</text>
        <text class="cal-value">1,200</text>
        <text>kcal</text>
      </view>
      <view class="data-item">
        <text>消耗</text>
        <text class="cal-value">850</text>
        <text>kcal</text>
      </view>
    </view>
  </view>
</template>

<script>

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