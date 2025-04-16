<template>
  <view class="search-container">
    <view class="search-box">
      <uni-icons type="search" size="26" color="#999" class="search-icon"></uni-icons>
      <input 
        class="search-input" 
        placeholder="请输入食物名称" 
        placeholder-class="placeholder-style" 
        v-model="searchText"
        @input="handleInput" 
         
      />
      <uni-icons type="scan" size="26" color="#9b9b9b" @click="goToCamera"></uni-icons> 
    </view>
    <text class="search-text" @click="performSearch">搜索</text>

    <view class="suggestion-list" v-if="showSuggestions && searchText">
      <view class="suggestion-item" v-for="(item, index) in suggestions" :key="index" @click="selectSuggestion(item)">
        <text class="suggestion-name">{{ item.name }}</text>
        <text class="suggestion-calories">{{ item.calories }}千卡/100克</text>
      </view>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  props: {
    initialKeyword: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      searchText: this.initialKeyword,
      showSuggestions: false,
      suggestions: [],
      searchTimer: null
    }
  },
  watch: {
    initialKeyword(newVal) {
      this.searchText = newVal
    }
  },
  methods: {
    goToCamera() {
      uni.navigateTo({
        url:'/pages/camera/camera',
      })
    },
    handleInput() {
      if (this.searchTimer) {
        clearTimeout(this.searchTimer)
      }

      this.searchTimer = setTimeout(async () => {
        if (this.searchText) {
          try {
            const response = await wx.cloud.callContainer({
              path: '/api/food/search', // 不需要带 BASE_URL
              method: 'GET',
              header: {
                'X-WX-SERVICE': '你的服务名', // ⚠️ 替换为你的云托管服务名
                'Authorization': 'Bearer ' + uni.getStorageSync('token'),
                'Content-Type': 'application/json'
              },
              // callContainer 的 GET 请求参数不能直接写 data，要拼接在 path 后面
              // 所以要用 `path` 拼接参数
              // 示例：/api/food/search?keyword=苹果
            })

            if (response.statusCode === 200) {
              this.suggestions = response.data
              this.showSuggestions = true
            } else {
              this.suggestions = []
              this.showSuggestions = false
            }
          } catch (err) {
            console.error('搜索失败：', err)
            this.suggestions = []
            this.showSuggestions = false
          }
        } else {
          this.suggestions = []
          this.showSuggestions = false
        }
      }, 300)
    },

    selectSuggestion(item) {
      this.searchText = item.name
      this.showSuggestions = false
      // 触发选择事件，将选中的食物传递给父组件
      this.$emit('select', item)
    },
    performSearch() {
      if (this.searchText) {
        // 跳转到食物列表页面，并传递搜索关键词
        uni.navigateTo({
          url: `/pages/food-list/food-list?keyword=${encodeURIComponent(this.searchText)}`,
          success: () => {
            // 清空搜索框
            this.searchText = ''
            this.showSuggestions = false
          }
        })
      }
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
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.suggestion-name {
  font-size: 28rpx;
  color: #333;
}

.suggestion-calories {
  font-size: 24rpx;
  color: #999;
}

.suggestion-item:last-child {
  border-bottom: none;
}

.suggestion-item:active {
  background-color: #f5f5f5;
}
</style>