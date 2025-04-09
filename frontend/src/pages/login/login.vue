<template>
  <view class="login-container">
    <view class="login-content">
      <image src="/static/logo.png" class="logo"></image>
      <text class="welcome-text">欢迎使用智食记录</text>

      <button class="wechat-login-btn" @click="goToHome">
        <uni-icons type="weixin" size="24" color="#fff"></uni-icons>
        <text>微信一键登录</text>
      </button>
    </view>
  </view>
</template>

<script>
export default {
  methods: {
    goToHome() {
      uni.switchTab({
        url: '/pages/home/home'
      })
    }
  }
}
</script>

<style scoped>
.login-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f8f8f8;
}

.login-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 80%;
}

.logo {
  width: 150rpx;
  height: 150rpx;
  margin-bottom: 40rpx;
  border-radius: 30rpx;
}

.welcome-text {
  font-size: 36rpx;
  color: #333;
  margin-bottom: 80rpx;
}

.wechat-login-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 90rpx;
  background-color: #07C160;
  color: #fff;
  border-radius: 45rpx;
  font-size: 32rpx;
  border: none;
}

.wechat-login-btn::after {
  border: none;
}

.wechat-login-btn text {
  margin-left: 15rpx;
}
</style>