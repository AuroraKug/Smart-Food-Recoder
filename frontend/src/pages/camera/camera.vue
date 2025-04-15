<template>
  <view class="camera-container">
    <!-- 相机预览区域 -->
    <camera
      class="camera-preview"
      :device-position="devicePosition"
      flash="off"
      @error="cameraError"
    ></camera>

    <!-- 底部操作栏 -->
    <view class="control-bar bottom">
      <!-- 相册按钮（左下角） -->
      <view class="btn-group left">
        <view class="btn album-btn" @tap="chooseImage">
          <uni-icons type="album" size="28" color="#fff"></uni-icons>
        </view>
      </view>
      
      <!-- 快门按钮（居中） -->
      <view class="btn-group center">
        <view class="shutter-btn" @tap="takePhoto"></view>
      </view>
      
      <!-- 切换按钮（右下角） -->
      <view class="btn-group right">
        <view class="btn switch-btn" @tap="switchCamera">
          <uni-icons type="loop" size="28" color="#fff"></uni-icons>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      devicePosition: 'back'
    }
  },
  methods: {
    takePhoto() {
      const ctx = uni.createCameraContext(this);
      ctx.takePhoto({
        quality: 'high',
        success: (res) => {
          const imagePath = res.tempImagePath;
          // 跳转到 identified-result 页面并传递图片路径
          uni.navigateTo({
            url: `/pages/identified-result/identified-result?imageURL=${encodeURIComponent(imagePath)}`
          });
        }
      });
    },
    switchCamera() {
      this.devicePosition = this.devicePosition === 'back' ? 'front' : 'back';
    },
    chooseImage() {
      uni.chooseImage({
        count: 1,
        sourceType: ['album'],
        success: (res) => {
          const imagePath = res.tempFilePaths[0];
          uni.navigateTo({
            url: `/pages/identified-result/identified-result?imagePath=${encodeURIComponent(imagePath)}`
          });
        }
      });
    },
    cameraError(e) {
      console.error('相机错误:', e.detail);
      uni.showToast({
        title: '相机初始化失败',
        icon: 'none'
      });
    }
  }
}
</script>

<style>
/* 保持原有样式不变 */
.camera-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.camera-preview {
  width: 100%;
  height: 100%;
}

/* 底部控制栏 - 核心布局 */
.control-bar.bottom {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 200rpx;
  display: flex;
  justify-content: space-between;
  background: rgba(0, 0, 0, 0.3);
  padding-bottom: calc(40rpx + env(safe-area-inset-bottom));
  z-index: 100;
}

/* 按钮组布局 */
.btn-group {
  flex: 1;
  display: flex;
  align-items: center;
  height: 100%;
  position: relative;
}

.btn-group.left {
  justify-content: flex-start;
  padding-left: 40rpx;
}

.btn-group.center {
  justify-content: center;
}

.btn-group.right {
  justify-content: flex-end;
  padding-right: 40rpx;
}

/* 按钮通用样式 */
.btn {
  width: 80rpx;
  height: 80rpx;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
}

/* 快门按钮特殊样式 */
.shutter-btn {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  border: 8rpx solid #fff;
  background: transparent;
  box-shadow: 0 0 0 4rpx rgba(255, 255, 255, 0.3);
  position: absolute;
  bottom: 20rpx;
}

/* iPad等大屏设备适配 */
@media (min-width: 768px) {
  .control-bar.bottom {
    height: 240rpx;
  }
  .shutter-btn {
    width: 140rpx;
    height: 140rpx;
    bottom: 30rpx;
  }
  .btn {
    width: 100rpx;
    height: 100rpx;
  }
  .btn-group.left {
    padding-left: 80rpx;
  }
  .btn-group.right {
    padding-right: 80rpx;
  }
}

/* 全面屏安全区域适配 */
@supports (bottom: env(safe-area-inset-bottom)) {
  .control-bar.bottom {
    padding-bottom: calc(60rpx + env(safe-area-inset-bottom));
  }
}
</style>