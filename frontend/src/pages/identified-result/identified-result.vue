<template>
  <view class="container">
    <image :src="capturedImagePath" class="background" mode="widthFix"></image>
    <view class="whiteboard"
      :style="{ height: whiteboardHeight + 'px', transition: 'height 0.3s cubic-bezier(0.42, 0, 0.58, 1)' }"
      @touchstart="handleTouchStart" @touchmove.stop="handleTouchMove" @touchend="handleTouchEnd">
      <view class="drag-handle"></view>

      <!-- 顶部信息区 -->
      <view class="top-section">
        <view class="food-header">
          <view class="icon-wrapper">
            <image src="/static/logo.png" class="food-icon" mode="aspectFit" />
          </view>

          <view class="food-info">
            <text class="food-name">Beer</text>

            <view class="progress-container">
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
      <text class="title-font">营养</text>
      <view class="nutrition-card"> 
        <view class="nutrition-row">
          <text class="label">热量</text>
          <view class="value-wrapper">
            <text class="value">65</text>
            <text class="unit">kcal/100ml</text>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      startY: 0,
      currentY: 0,
      whiteboardHeight: 0,
      minHeight: 0,
      maxHeight: 0,
      capturedImagePath: '',
      progress: 65, // 默认进度值
    };
  },
  onLoad(options) {
    this.capturedImagePath = options.imagePath ? decodeURIComponent(options.imagePath) : '';
    const systemInfo = uni.getSystemInfoSync();
    const screenHeight = systemInfo.windowHeight;
    this.minHeight = Math.floor(screenHeight * 0.3);
    this.maxHeight = Math.floor(screenHeight * 0.85);
    this.whiteboardHeight = this.minHeight;
  },
  methods: {
    handleTouchStart(event) {
      this.startY = event.touches[0].pageY;
      this.currentY = this.startY;
    },
    handleTouchMove(event) {
      const moveY = event.touches[0].pageY;
      const diffY = this.startY - moveY;
      this.whiteboardHeight = Math.max(this.minHeight, Math.min(this.maxHeight, this.whiteboardHeight + diffY));
      this.currentY = moveY;
    },
    handleTouchEnd() {
      const lastDirection = this.currentY - this.startY;
      if (lastDirection < 0) {
        this.whiteboardHeight = this.maxHeight;
      } else {
        this.whiteboardHeight = this.minHeight;
      }
    },
    getProgressColor(percent) {
      // 更平滑的HSL颜色过渡（红0° → 绿120°）
      const hue = (1 - percent / 100) * 120;
      return `hsl(${hue}, 100%, 50%)`;
    }
  }
}
</script>

<style scoped>
/* 保持原有样式不变 */
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
  padding: 20rpx;
}

.nutrition-card {
  background: #F8F9FA;
  border-radius: 16rpx;
  padding: 30rpx;
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
</style>