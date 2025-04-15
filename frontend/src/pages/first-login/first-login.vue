<template>
  <view class="first-login-container">
    <view class="header">
      <text class="title">完善个人信息</text>
      <text class="subtitle">请填写以下信息以开始使用</text>
    </view>

    <view class="form-container">
      <view class="input-item">
        <text class="label">用户名</text>
        <input class="input" v-model="userInfo.username" placeholder="请输入用户名" />
      </view>

      <view class="input-item" @click="editField('gender')">
        <text class="label">性别</text>
        <text class="value">{{ userInfo.gender }}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>

      <view class="input-item" @click="openPicker('birthdate')">
        <text class="label">出生日期</text>
        <text class="value">{{ userInfo.birthdate}}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>

      <view class="input-item" @click="$refs.heightKeyboard.open(userInfo.height)">
        <text class="label">身高</text>
        <text class="value">{{ displayHeight }}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>

      <view class="input-item" @click="$refs.currentWeightKeyboard.open(userInfo.currentWeight)">
        <text class="label">当前体重</text>
        <text class="value">{{ displayCurrentWeight }}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>

      <view class="input-item" @click="$refs.targetWeightKeyboard.open(userInfo.targetWeight)">
        <text class="label">目标体重</text>
        <text class="value">{{ displayTargetWeight }}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>
    </view>

    <button class="submit-btn" @click="submitInfo">完成</button>

    <DatePicker ref="datePicker" :field="currentField" :min-year="1900" :max-year="new Date().getFullYear()"
       @save="handleDateSave" />

    <NumberKeyboard ref="heightKeyboard" field="height" title="请输入身高" unit="cm" :max-length="5" :range="[50, 250]"
      :decimal-digits="1" @confirm="handleNumberConfirm" />

    <NumberKeyboard ref="currentWeightKeyboard" field="currentWeight" title="请输入当前体重" unit="kg" :max-length="5"
      :range="[30, 200]" :decimal-digits="1" @confirm="handleNumberConfirm" />

    <NumberKeyboard ref="targetWeightKeyboard" field="targetWeight" title="请输入目标体重" unit="kg" :max-length="5"
      :range="[30, 200]" :decimal-digits="1" @confirm="handleNumberConfirm" />
  </view>
</template>

<script>
import NumberKeyboard from '@/components/NumberKeyboard.vue'
import DatePicker from '@/components/DatePicker.vue'

const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  components: {
    NumberKeyboard,
    DatePicker
  },
  data() {
    return {
      userInfo: {
        username: '',
        gender: '',
        birthdate: '',
        height: '',
        currentWeight: '',
        targetWeight: ''
      },
      currentField: ''
    }
  },
  computed: {
    displayHeight() {
      return this.userInfo.height ? `${this.userInfo.height}cm` : ''
    },
    displayCurrentWeight() {
      return this.userInfo.currentWeight ? `${this.userInfo.currentWeight}kg` : ''
    },
    displayTargetWeight() {
      return this.userInfo.targetWeight ? `${this.userInfo.targetWeight}kg` : ''
    }
  },
  methods: {
    editField(field) {
      if (field === 'gender') {
        uni.showActionSheet({
          itemList: ['男', '女'],
          success: (res) => {
            if (res.tapIndex !== undefined) {
              this.userInfo.gender = res.tapIndex === 0 ? '男' : '女' 
            }
          }
        }) 
      }
    },
    openPicker(field) {
      this.currentField = field
      this.$refs.datePicker.showPicker = true
    },
    handleDateSave({ field, date }) {
      if (field === 'birthdate') {
        this.userInfo.birthdate = date
      }
    },
    handleNumberConfirm({ field, value }) {
      if (field === 'height') {
        this.userInfo.height = value
      } else if (field === 'currentWeight') {
        this.userInfo.currentWeight = value
      } else if (field === 'targetWeight') {
        this.userInfo.targetWeight = value
      }
    },
    async submitInfo() {
      if (!this.userInfo.username) {
        uni.showToast({
          title: '请输入用户名',
          icon: 'none'
        })
        return
      }

      if (!this.userInfo.birthdate) {
        uni.showToast({
          title: '请选择出生日期',
          icon: 'none'
        })
        return
      }

      if (!this.userInfo.height) {
        uni.showToast({
          title: '请输入身高',
          icon: 'none'
        })
        return
      }

      if (!this.userInfo.currentWeight) {
        uni.showToast({
          title: '请输入当前体重',
          icon: 'none'
        })
        return
      }

      if (!this.userInfo.targetWeight) {
        uni.showToast({
          title: '请输入目标体重',
          icon: 'none'
        })
        return
      }

      try {
        // 更新用户基本信息
        const userUpdateResponse = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/user/update',
            method: 'POST',
            data: {
              userName: this.userInfo.username,
              birthdate: this.userInfo.birthdate,
              gender: this.userInfo.gender,
              height: this.userInfo.height
            },
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

        if (userUpdateResponse.statusCode !== 200) {
          throw new Error(userUpdateResponse.data.message || '保存用户信息失败')
        }

        // 保存体重相关信息
        const weightResponse = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/weight/goal/save',
            method: 'POST',
            data: {
              startWeight: parseFloat(this.userInfo.currentWeight),
              currentWeight: parseFloat(this.userInfo.currentWeight),
              targetWeight: parseFloat(this.userInfo.targetWeight),
            },
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

        if (weightResponse.statusCode === 200) {
          uni.showToast({
            title: '信息保存成功',
            icon: 'success'
          })
          
          // 跳转到首页
          uni.switchTab({
            url: '/pages/home/home'
          })
        } else {
          throw new Error(weightResponse.data.message || '保存体重信息失败')
        }
      } catch (err) {
        console.error('保存信息失败：', err)
        uni.showToast({
          title: err.message || '保存失败',
          icon: 'none'
        })
      }
    }
  }
}
</script>

<style scoped>
.first-login-container {
  padding: 40rpx;
  min-height: 100vh;
  background-color: #f8f8f8;
}

.header {
  margin-bottom: 60rpx;
  text-align: center;
}

.title {
  font-size: 40rpx;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 20rpx;
}

.subtitle {
  font-size: 28rpx;
  color: #666;
}

.form-container {
  background-color: #fff;
  border-radius: 20rpx;
  padding: 0 30rpx;
  margin-bottom: 40rpx;
}

.input-item {
  display: flex;
  align-items: center;
  padding: 30rpx 0;
  border-bottom: 1rpx solid #eee;
}

.input-item:last-child {
  border-bottom: none;
}

.label {
  font-size: 32rpx;
  color: #333;
  width: 160rpx;
}

.input {
  flex: 1;
  font-size: 32rpx;
  color: #333;
  text-align: left;
  padding-left: 0;
}

.input::placeholder {
  color: #999;
  text-align: left;
}

.value {
  flex: 1;
  font-size: 32rpx;
  color: #666;
  text-align: left;
  padding-right: 20rpx;
}

.submit-btn {
  width: 100%;
  height: 88rpx;
  line-height: 88rpx;
  background-color: #4cd964;
  color: #fff;
  font-size: 32rpx;
  border-radius: 44rpx;
  margin-top: 60rpx;
}
</style> 