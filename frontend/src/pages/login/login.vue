<template>
  <view class="login-container">
    <view class="login-content">
      <image src="/static/logo.png" class="logo"></image>
      <text class="welcome-text">欢迎使用智食记录</text>

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
          });
        });

        console.log('微信code:', loginRes.code);

        // 用 Promise 包装 uni.request，解决不能 await 的问题
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com/api/auth/login',
            method: 'POST',
            data: {
              code: loginRes.code
            },
            header: {
              'Content-Type': 'application/json'
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          });
        });

        if (response.statusCode === 200) {
          const {token, openid ,if_first_login} = response.data;

          uni.setStorageSync('token', token);
          uni.setStorageSync('userInfo', openid);

          uni.showToast({
            title: '登录成功'
          });

          // uni.switchTab({
          //   url: '/pages/home/home'
          // });
          // uni.navigateTo({
          //   url: '/pages/first-login/first-login'
          // });
          if (if_first_login === true) {
    // 首次登录，跳转到首次信息填写页
            uni.navigateTo({
              url: '/pages/first-login/first-login'
            });
          } else {
            // 非首次登录，跳转到首页
            uni.switchTab({
              url: '/pages/home/home'
            });
          }

        } else {
          throw new Error(response.data.message || '登录失败');
        }
      } catch (err) {
        console.error('登录出错:', err);
        uni.showToast({
          title: err.message || '登录失败',
          icon: 'none'
        });
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