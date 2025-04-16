<template>
  <view class="login-container">
    <view class="login-content">
      <image src="/static/logo.png" class="logo"></image>
      <text class="welcome-text">欢迎使用智食记</text>

      <button class="wechat-login-btn" @click="login">
        <uni-icons type="weixin" size="24" color="#fff"></uni-icons>
        <text>微信一键登录</text>
      </button>
    </view>
  </view>
</template>

<script>
export default {
  methods: {
    async login() {
      try {
        const loginRes = await new Promise((resolve, reject) => {
          uni.login({
            provider: 'weixin',
            success: resolve,
            fail: reject
          })
        })

        const response = await wx.cloud.callContainer({
          path: '/api/auth/login', // 不需要带域名
          method: 'POST',
          data: {
            code: loginRes.code
          },
          header: {
            'X-WX-SERVICE': 'springboot-glwv', // 注意替换成你实际云托管服务名称（在控制台云托管服务详情可查）
            'Content-Type': 'application/json'
          }
        })

        if (response.statusCode === 200) {
          const {token, openid ,if_first_login} = response.data;

          uni.setStorageSync('token', token);
          uni.setStorageSync('userInfo', openid);

          uni.showToast({
            title: '登录成功'
          })

          if (if_first_login === true) {
            uni.navigateTo({
              url: '/pages/first-login/first-login'
            })
          } else {
            uni.switchTab({
              url: '/pages/home/home'
            })
          }

        } else {
          throw new Error(response.data.message || '登录失败')
        }
      } catch (err) {
        console.error('登录出错:', err)
        uni.showToast({
          title: err.message || '登录失败',
          icon: 'none'
        })
      }
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
  background-color: #fff;
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