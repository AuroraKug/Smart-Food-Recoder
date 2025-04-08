<template>
  <view class="recent-records">
    <view class="section-title">
      <text>最近记录</text>
      <text class="more">查看更多 ></text>
    </view>
    <view class="record-list">
      <view class="record-item">
        <text>早餐</text>
        <text>350 kcal</text>
      </view>
      <view class="record-item">
        <text>午餐</text>
        <text>580 kcal</text>
      </view>
    </view>
  </view>
</template>

<script>

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
</style>