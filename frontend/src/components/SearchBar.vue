<template>
  <view class="search-container">
    <view class="search-box">
      <uni-icons type="search" size="18" color="#999" class="search-icon"></uni-icons>
      <input 
        class="search-input" 
        placeholder="请输入搜索内容" 
        placeholder-class="placeholder-style" 
        v-model="searchText"
        @input="handleInput" 
        focus 
      />
      <uni-icons type="scan" size="26" color="#9b9b9b" @click="goToCamera"></uni-icons> 
    </view>
    <text class="search-text" @click="performSearch">搜索</text>

    <view class="suggestion-list" v-if="showSuggestions && searchText">
      <view class="suggestion-item" v-for="(item, index) in suggestions" :key="index" @click="selectSuggestion(item)">
        {{ item }}
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      searchText: '',
      showSuggestions: false,
      suggestions: []
    }
  },
  methods: {
    goToCamera() {
      uni.navigateTo({
        url:'/pages/camera/camera',
      })
    },
    handleInput() {
      if (this.searchText) {
        this.showSuggestions = true
        this.suggestions = ['苹果', '香蕉', '鸡蛋', '面包'].filter(item =>
          item.includes(this.searchText)
        )
      } else {
        this.showSuggestions = false
        this.suggestions = []
      }
    },
    selectSuggestion(item) {
      this.searchText = item
      this.showSuggestions = false
    },
    performSearch() {
      console.log('搜索:', this.searchText)
      this.showSuggestions = false
    }
  }
}
</script>

<style scoped>
.search-container {
  padding: 15rpx 30rpx;
  background-color: transparent; 
  position: relative;
}

.search-box {
  display: flex;
  align-items: center;
  height: 70rpx;
  background-color: transparent; 
  border: 2rpx solid #e0e0e0; 
  border-radius: 35rpx;
  padding: 0 20rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
  width: 80%;
}

.search-icon {
  margin-right: 10rpx;
}

.search-input {
  flex: 1;
  height: 100%;
  font-size: 28rpx;
  color: #333;
  background-color: transparent;
}

.placeholder-style {
  color: #999;
  font-size: 28rpx;
}

.scan-placeholder {
  width: 40rpx; 
  height: 40rpx;
  margin-left: 10rpx;
}

.search-text {
  position: absolute;
  right: 30rpx;
  top: 20rpx;
  font-size: 32rpx;
  color: #4cd964; 
  cursor: pointer;
  line-height: 70rpx;
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