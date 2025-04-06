<template>
    <view class="login-container">
      <!-- 登录页面内容 -->
      <view class="login-content">
        <image src="/static/logo.png" class="logo"></image>
        <text class="welcome-text">欢迎使用智食记录</text>
        
        <!-- 微信登录按钮 -->
        <button 
          class="wechat-login-btn"
          open-type="getUserInfo"
          @getuserinfo="onGetUserInfo"
        >
          <uni-icons type="weixin" size="24" color="#fff"></uni-icons>
          <text>微信一键登录</text>
        </button>
      </view>
    </view>
  </template>
  
  <script>
  export default {
    methods: {
      // 获取用户信息
      onGetUserInfo(e) {
        if (e.detail.userInfo) {
          // 用户同意授权
          this.wechatLogin(e.detail)
        } else {
          // 用户拒绝授权
          uni.showToast({
            title: '您拒绝了授权',
            icon: 'none'
          })
        }
      },
      
      // 微信登录
      wechatLogin(userInfo) {
        uni.showLoading({
          title: '登录中...'
        })
        
        // 1. 获取code
        uni.login({
          provider: 'weixin',
          success: (loginRes) => {
            if (loginRes.code) {
              // 2. 发送code和用户信息到后端
              uni.request({
                url: '你的后端登录接口',
                method: 'POST',
                data: {
                  code: loginRes.code,
                  userInfo: userInfo
                },
                success: (res) => {
                  uni.hideLoading()
                  if (res.data.success) {
                    // 登录成功处理
                    this.loginSuccess(res.data)
                  } else {
                    uni.showToast({
                      title: res.data.message || '登录失败',
                      icon: 'none'
                    })
                  }
                },
                fail: () => {
                  uni.hideLoading()
                  uni.showToast({
                    title: '网络请求失败',
                    icon: 'none'
                  })
                }
              })
            } else {
              uni.hideLoading()
              uni.showToast({
                title: '获取登录凭证失败',
                icon: 'none'
              })
            }
          },
          fail: () => {
            uni.hideLoading()
            uni.showToast({
              title: '微信登录失败',
              icon: 'none'
            })
          }
        })
      },
      
      // 登录成功处理
      loginSuccess(data) {
        // 保存用户信息和token
        uni.setStorageSync('userInfo', data.userInfo)
        uni.setStorageSync('token', data.token)
        uni.setStorageSync('hasLogin', true)
        
        // 跳转到首页
        uni.switchTab({
          url: '/pages/index/index'
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