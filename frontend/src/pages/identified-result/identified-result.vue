<template>
  <view class="container">
    <image :src="capturedImagePath" class="background" mode="widthFix"></image>
    <view class="whiteboard"
      :style="{ height: whiteboardHeight + 'px', transition: 'height 0.3s cubic-bezier(0.42, 0, 0.58, 1)' }"
      @touchstart="handleTouchStart" @touchmove.stop="handleTouchMove" @touchend="handleTouchEnd">
      <view class="drag-handle"></view>

      <!-- 顶部信息区 -->
      <view class="top-section" v-if="!mainResult || (mainResult && mainResult.name !== '非菜')">
        <view class="food-header">
          <view class="icon-wrapper">
            <image :src="capturedImagePath" class="food-icon" mode="aspectFit" />
          </view>

          <view class="food-info">
            <text class="food-name" v-if="!mainResult">识别中...</text>
            <text class="food-name" v-else>{{ mainResult.name }}</text>

            <view class="progress-container" v-if="mainResult && mainResult.name !== '非菜'">
              <uni-icons type="eye" size="18" color="#999"></uni-icons>
              <view class="progress-bar">
                <view class="progress-fill" :style="{
                  width: progress + '%',
                  background: getProgressColor(progress)
                }"></view>
              </view>
              <text class="progress-value">{{ progress }}%</text>
            </view>
          </view>
        </view>
      </view>

      <!-- 营养卡片 -->
      <text class="title-font" v-if="mainResult && mainResult.name !== '非菜'">营养</text>
      <view class="nutrition-card" v-if="mainResult && mainResult.name !== '非菜'"> 
        <view class="nutrition-row">
          <text class="label">热量</text>
          <view class="value-wrapper">
            <text class="value">{{ mainResult ? mainResult.calorie : '--' }}</text>
            <text class="unit">卡/100克</text>
          </view>
        </view>
      </view>

      <!-- 其他可能结果 -->
      <text class="title-font" v-if="otherResults.length > 0 && mainResult && mainResult.name !== '非菜'">其他可能结果</text>
      <view class="other-results" v-if="otherResults.length > 0 && mainResult && mainResult.name !== '非菜'">
        <view class="other-result-item" v-for="(item, index) in otherResults" :key="index">
          <text class="other-food-name">{{ item.name }}</text>
          <text class="other-food-calories">{{ item.calorie }}卡/100克</text>
          <text class="other-probability">{{ (item.probability * 100).toFixed(2) }}%</text>
        </view>
      </view>

      <!-- 非菜提示 -->
      <view class="non-food-warning" v-if="mainResult && mainResult.name === '非菜'">
        <text class="warning-text">⚠️ 识别结果可能不是食物</text>
        <text class="warning-desc">请尝试重新拍摄或选择其他食物</text>
      </view>

      <!-- 操作按钮 -->
      <view class="button-container">
        <button class="action-button" @click="retakePhoto">重新拍摄</button>
        <button class="action-button" @click="confirmSelection" v-if="mainResult && mainResult.name !== '非菜'">确认选择</button>
      </view>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  data() {
    return {
      startY: 0,
      currentY: 0,
      whiteboardHeight: 0,
      minHeight: 0,
      maxHeight: 0,
      capturedImagePath: '',
      progress: 0,
      mainResult: null,
      otherResults: [],
      hasLowProbability: false
    }
  },
  onLoad(options) {
    this.capturedImagePath = options.imagePath ? decodeURIComponent(options.imagePath) : ''
    const systemInfo = uni.getSystemInfoSync()
    const screenHeight = systemInfo.windowHeight
    this.minHeight = Math.floor(screenHeight * 0.3)
    this.maxHeight = Math.floor(screenHeight * 0.85)
    this.whiteboardHeight = this.minHeight
    this.identifyFood()
  },
  methods: {
    handleTouchStart(event) {
      this.startY = event.touches[0].pageY
      this.currentY = this.startY
    },
    handleTouchMove(event) {
      const moveY = event.touches[0].pageY
      const diffY = this.startY - moveY
      this.whiteboardHeight = Math.max(this.minHeight, Math.min(this.maxHeight, this.whiteboardHeight + diffY))
      this.currentY = moveY
    },
    handleTouchEnd() {
      const lastDirection = this.currentY - this.startY
      if (lastDirection < 0) {
        this.whiteboardHeight = this.maxHeight
      } else {
        this.whiteboardHeight = this.minHeight
      }
    },
    getProgressColor(percent) {
      const hue = (1 - percent / 100) * 120
      return `hsl(${hue}, 100%, 50%)`
    },
    async identifyFood() {
      try {
        const response = await new Promise((resolve, reject) => {
          uni.uploadFile({
            url: BASE_URL + '/api/photo',
            filePath: this.capturedImagePath,
            name: 'file',
            header: {
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

        if (response.statusCode === 200) {
          const data = JSON.parse(response.data)
          if (data.result && data.result.length > 0) {
            // 设置主要结果
            this.mainResult = data.result[0]
            this.progress = (this.mainResult.probability * 100).toFixed(2)
            
            // 过滤其他结果（概率大于10%）
            this.otherResults = data.result.slice(1).filter(item => 
              parseFloat(item.probability) > 0.1
            )
            
            // 检查是否有低概率结果
            this.hasLowProbability = data.result.some(item => 
              parseFloat(item.probability) <= 0.1
            )
          }
        }
      } catch (err) {
        console.error('识别失败：', err)
        uni.showToast({
          title: '识别失败',
          icon: 'none'
        })
      }
    },
    retakePhoto() {
      uni.navigateBack()
    },
    confirmSelection() {
      if (this.mainResult) {
        this.$emit('select', this.mainResult)
        uni.navigateBack()
      }
    }
  }
}
</script>

<style scoped>
.container {
  position: relative;
  height: 100vh;
  overflow: hidden;
}

.background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-repeat: no-repeat;
  z-index: 1;
  touch-action: none;
}

.whiteboard {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background-color: #f2f2f2;
  border-top-left-radius: 20rpx;
  border-top-right-radius: 20rpx;
  z-index: 2;
  box-shadow: 0 -2rpx 10rpx rgba(0, 0, 0, 0.1);
  touch-action: none;
  padding: 30rpx;
  box-sizing: border-box;
}

.drag-handle {
  width: 80rpx;
  height: 8rpx;
  background-color: #333;
  border-radius: 4rpx;
  position: absolute;
  top: 20rpx;
  left: 50%;
  transform: translateX(-50%);
}

.top-section {
  height: 200rpx;
  margin-bottom: 40rpx;
}

.food-header {
  display: flex;
  align-items: center;
  height: 100%;
}

.icon-wrapper {
  width: 120rpx;
  height: 120rpx;
  background: #FFF4E6;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 30rpx;
}

.food-icon {
  width: 80rpx;
  height: 80rpx;
}

.food-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.food-name {
  font-size: 36rpx;
  font-weight: 500;
  color: #333;
  margin-bottom: 12rpx;
}

.progress-container {
  display: flex;
  align-items: center;
  margin-top: 15rpx;
}

.progress-bar {
  flex: 1;
  height: 8rpx;
  background: #b3b3b3;
  border-radius: 4rpx;
  margin: 0 15rpx;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  border-radius: 4rpx;
  transition: all 0.3s ease;
}

.progress-value {
  font-size: 24rpx;
  color: #666;
}

.title-font {
  font-size: 32rpx;
  color: #999999;
  padding: 20rpx 0;
}

.nutrition-card {
  background: #F8F9FA;
  border-radius: 16rpx;
  padding: 30rpx;
  margin-bottom: 20rpx;
}

.nutrition-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20rpx 0;
}

.label {
  font-size: 30rpx;
  color: #666;
}

.value-wrapper {
  display: flex;
  align-items: baseline;
}

.value {
  font-size: 36rpx;
  color: #4CAF50;
  font-weight: bold;
  margin-right: 10rpx;
}

.unit {
  font-size: 24rpx;
  color: #999;
}

.other-results {
  background: #F8F9FA;
  border-radius: 16rpx;
  padding: 20rpx;
  margin-bottom: 20rpx;
}

.other-result-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15rpx 0;
  border-bottom: 1rpx solid #eeeeee;
}

.other-result-item:last-child {
  border-bottom: none;
}

.other-food-name {
  font-size: 28rpx;
  color: #333333;
}

.other-food-calories {
  font-size: 24rpx;
  color: #666666;
}

.other-probability {
  font-size: 24rpx;
  color: #999999;
}

.non-food-warning {
  background-color: #fff3f3;
  border-radius: 12rpx;
  padding: 30rpx;
  margin: 40rpx 0;
  text-align: center;
}

.warning-text {
  font-size: 32rpx;
  color: #ff4d4f;
  font-weight: bold;
  margin-bottom: 10rpx;
  display: block;
}

.warning-desc {
  font-size: 28rpx;
  color: #666;
  display: block;
  margin-top: 10rpx;
}

.button-container {
  display: flex;
  justify-content: space-around;
  margin-top: 40rpx;
}

.action-button {
  width: 45%;
  height: 80rpx;
  line-height: 80rpx;
  text-align: center;
  border-radius: 40rpx;
  font-size: 32rpx;
  color: #ffffff;
  background-color: #4cd964;
}

.action-button:active {
  opacity: 0.8;
}
</style>