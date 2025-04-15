<template>
  <view class="profile-container">
    <!-- 用户信息卡片 -->
    <view class="profile-card">
      <view class="avatar-section">
        <image class="avatar" :src="userInfo.avatar"></image>
      </view>
      <view class="info-section">
        <text class="name">{{ userInfo.nickname }}</text>
      </view>
    </view>

    <!-- 设置列表 -->
    <view class="settings-list">
      <view class="list-item" @click="goToInfo">
        <uni-icons type="person" size="18" color="#666"></uni-icons>
        <text>个人信息设置</text>
        <uni-icons type="forward" size="14" color="#999"></uni-icons>
      </view>
      <view class="list-item" @click="goToRecord">
        <uni-icons type="list" size="18" color="#666"></uni-icons>
        <text>饮食记录</text>
        <uni-icons type="forward" size="14" color="#999"></uni-icons>
      </view>
      <view class="list-item" @click="goToHistory">
        <uni-icons type="star" size="18" color="#666"></uni-icons>
        <text>拍照历史</text>
        <uni-icons type="forward" size="14" color="#999"></uni-icons>
      </view>
    </view>

    <!-- 退出登录 -->
    <view class="logout-btn" @click="logout">
      <text>退出登录</text>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  data() {
    return {
      userInfo: {
        avatar: '',
        nickname: ''
      },
      userData: {
        weight: 62.5,
        days: 28,
        achievements: 3
      }
    }
  },
  onLoad() {
    this.getUserProfile()
  },
  onPullDownRefresh() {
    this.getUserProfile().finally(() => {
      uni.stopPullDownRefresh()
    })
  },
  methods: {
    // 获取用户信息
    async getUserProfile() {
      try {
        // 获取后端用户信息
        const res = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/user/info',
            method: 'GET',
            header: {
              'Authorization': 'Bearer ' + uni.getStorageSync('token'),
              'Content-Type': 'application/json'
            },
            success: resolve,
            fail: reject
          });
        });

        if (res.statusCode === 200) {
          this.userInfo.nickname = res.data.userName
        }

        // 检查是否已授权
        uni.getSetting({
          success: (res) => {
            if (res.authSetting['scope.userInfo']) {
              // 已经授权，可以直接调用 getUserInfo 获取头像
              this.getUserInfo()
            } else {
              // 未授权，调用微信登录接口
              uni.getUserProfile({
                desc: '用于完善用户资料',
                success: (res) => {
                  this.userInfo.avatar = res.userInfo.avatarUrl
                },
                fail: (err) => {
                  console.error('获取用户信息失败', err)
                  uni.showToast({
                    title: '获取用户信息失败',
                    icon: 'none'
                  })
                }
              })
            }
          }
        })
      } catch (err) {
        console.error('获取用户信息失败：', err);
        uni.showToast({
          title: '获取用户信息失败',
          icon: 'none'
        });
      }
    },

    // 获取已授权的用户信息
    getUserInfo() {
      uni.getUserInfo({
        success: (res) => {
          this.userInfo.avatar = res.userInfo.avatarUrl
        }
      })
    },

    goToInfo() {
      uni.navigateTo({
        url:'/pages/profile-info/profile-info',
      })
    },
    goToRecord() {
      uni.navigateTo({
        url:'/pages/food-record/food-record',
      })
    },
    goToHistory() {
      uni.navigateTo({
        url:'/pages/photo-history/photo-history',
      })
    },
    logout() {
      uni.showModal({
        title: '提示',
        content: '确定要退出登录吗？',
        success: (res) => {
          if (res.confirm) {
            uni.reLaunch({ url: '/pages/login/login' })
          }
        }
      })
    }
  }
}
</script>

<style>
.profile-container {
  position: fixed; 
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 20rpx;
  background-color: #f5f5f5;
  overflow-y: auto; 
  z-index: 0; 
}

.profile-card {
  background: #fff;
  border-radius: 16rpx;
  padding: 40rpx;
  display: flex;
  align-items: center;
  margin-bottom: 20rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
}

.avatar-section {
  margin-right: 30rpx;
}

.avatar {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  border: 4rpx solid #f0f0f0;
}

.info-section {
  flex: 1;
}

.name {
  font-size: 36rpx;
  font-weight: bold;
  display: block;
}

.data-card {
  background: #fff;
  border-radius: 16rpx;
  padding: 30rpx 0;
  display: flex;
  justify-content: space-around;
  margin-bottom: 20rpx;
}

.data-item {
  text-align: center;
}

.data-item .value {
  font-size: 40rpx;
  font-weight: bold;
  color: #4CD964;
  display: block;
}

.data-item .label {
  font-size: 24rpx;
  color: #999;
  margin-top: 8rpx;
}

.settings-list {
  background: #fff;
  border-radius: 16rpx;
  overflow: hidden;
}

.list-item {
  padding: 30rpx;
  display: flex;
  align-items: center;
  border-bottom: 1rpx solid #f0f0f0;
}

.list-item text {
  flex: 1;
  margin-left: 20rpx;
  font-size: 28rpx;
}

.logout-btn {
  margin-top: 40rpx;
  background: #fff;
  color: #ff4d4f;
  text-align: center;
  padding: 30rpx;
  border-radius: 16rpx;
  font-size: 30rpx;
}
</style>