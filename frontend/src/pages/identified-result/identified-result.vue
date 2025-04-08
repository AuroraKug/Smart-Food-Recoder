<template>
  <view class="container">
    <image :src="capturedImagePath" class="background" mode="widthFix"></image>
    <view 
      class="whiteboard" 
      :style="{ height: whiteboardHeight + 'px', transition: 'height 0.3s cubic-bezier(0.42, 0, 0.58, 1)' }" 
      @touchstart="handleTouchStart" 
      @touchmove.stop="handleTouchMove" 
      @touchend="handleTouchEnd"
    >
      <view class="drag-handle"></view>
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
    };
  },
  onLoad(options) {
    this.capturedImagePath = options.imagePath ? decodeURIComponent(options.imagePath) : '默认图片路径';
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
  },
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
  background-color: #ffffff;
  border-top-left-radius: 20rpx;
  border-top-right-radius: 20rpx;
  z-index: 2;
  box-shadow: 0 -2rpx 10rpx rgba(0, 0, 0, 0.1);
  touch-action: none;
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
</style>