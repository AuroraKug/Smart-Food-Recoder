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
    login() {
      uni.login({
        provider: 'weixin',
        success: (res) => {
          console.log('登录成功, code: ', res.code);
          uni.switchTab({
            url: '/pages/home/home',
          })
        },
        fail: (err) => {
          console.error('登录失败: ', err)
          uni.showToast({
            title: '登录失败'
          })
        }
      })
    }
  }
  // async login() {
  //   try {
  //     // 1. 获取微信 code（注意：uni.login() 返回的是对象，不是数组）
  //     const loginRes = await uni.login({
  //       provider: 'weixin'
  //     });
  //     console.log('微信code:', loginRes.code); // 正确访问 code

  //     // 2. 发送 code 到后端
  //     const [err, res] = await uni.request({
  //       url: 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com/api/auth/login',
  //       method: 'POST',
  //       data: {
  //         code: loginRes.code
  //       },
  //       header: {
  //         'Content-Type': 'application/json'
  //       }
  //     });

  //     // 3. 处理错误
  //     if (err) {
  //       throw new Error(err.errMsg || '请求失败');
  //     }

  //     // 4. 处理后端响应
  //     if (res.statusCode === 200) {
  //       const { token, userInfo } = res.data;

  //       // 存储 Token
  //       uni.setStorageSync('token', token);

  //       // 跳转到首页
  //       uni.switchTab({
  //         url: '/pages/home/home'
  //       });
  //     } else {
  //       throw new Error(res.data.message || '登录失败');
  //     }
  //   } catch (err) {
  //     console.error('登录出错:', err);
  //     uni.showToast({
  //       title: err.message || '登录失败',
  //       icon: 'none'
  //     });
  //   }
  // }
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