<template>
  <view class="search-container">
    <view class="search-box">
      <uni-icons type="search" size="18" color="#999" class="search-icon"></uni-icons>
      <input class="search-input" placeholder="请输入搜索内容" placeholder-class="placeholder-style" v-model="searchText"
        @input="handleInput" focus />
    </view>

    <view class="suggestion-list" v-if="showSuggestions && searchText">
      <view class="suggestion-item" v-for="(item, index) in suggestions" :key="index" @click="selectSuggestion(item)">
        {{ item }}
      </view>
    </view>
  </view>
</template>

<script>

</script>

<style scoped>
.search-container {
  padding: 15rpx 30rpx;
  background-color: #f8f8f8;
  position: relative;
}

.search-box {
  display: flex;
  align-items: center;
  height: 70rpx;
  background-color: #fff;
  border-radius: 35rpx;
  padding: 0 20rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.search-icon {
  margin-right: 10rpx;
}

.search-input {
  flex: 1;
  height: 100%;
  font-size: 28rpx;
  color: #333;
}

.placeholder-style {
  color: #999;
  font-size: 28rpx;
}

.suggestion-list {
  position: absolute;
  left: 30rpx;
  right: 30rpx;
  top: 100rpx;
  background-color: #fff;
  border-radius: 10rpx;
  box-shadow: 0 4rpx 12rpx #b3b3b3;
  z-index: 100;
  max-height: 500rpx;
  overflow-y: auto;
}

.suggestion-item {
  padding: 20rpx 30rpx;
  font-size: 28rpx;
  color: #333;
  border-bottom: 1rpx solid #cccccc;
}

.suggestion-item:last-child {
  border-bottom: none;
}

.suggestion-item:active {
  background-color: #f5f5f5;
}
</style>