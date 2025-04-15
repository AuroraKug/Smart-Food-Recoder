<template>
  <view class="page-wrapper">
    <view class="profile-container">
      <view class="profile-block">
        <view class="profile-item" @tap="editField('username')">
          <text class="label">用户名</text>
          <text class="value">{{ profileData.username }}</text>
          <uni-icons type="forward" size="16" color="#999"></uni-icons>
        </view>
        <view class="profile-item" @tap="editField('gender')">
          <text class="label">性别</text>
          <text class="value">{{ profileData.gender }}</text>
          <uni-icons type="forward" size="16" color="#999"></uni-icons>
        </view>
        <view class="profile-item" @click="openPicker('birth')">
          <text class="label">生日</text>
          <text class="value">{{ profileData.birthdate }}</text>
          <uni-icons type="forward" size="16" color="#999"></uni-icons>
        </view>
        <view class="profile-item" @click="$refs.heightKeyboard.open(parseFloat(profileData.height) || '')">
          <text class="label">身高</text>
          <text class="value">{{ tempValues.height || (profileData.height ? profileData.height + 'cm' : '') }}</text>
          <uni-icons type="forward" size="16" color="#999"></uni-icons>
        </view>
      </view>

      <view class="profile-block">
        <view class="profile-item" @click="$refs.initialWeightKeyboard.open(parseFloat(profileData.initialWeight) || '')">
          <text class="label">初始体重</text>
          <text class="value">{{ tempValues.initialWeight || (profileData.initialWeight ? profileData.initialWeight + 'kg' : '') }}</text>
          <uni-icons type="forward" size="16" color="#999"></uni-icons>
        </view>
        <view class="profile-item" @click="$refs.currentWeightKeyboard.open(parseFloat(profileData.currentWeight) || '')">
          <text class="label">当前体重</text>
          <text class="value">{{ tempValues.currentWeight || (profileData.currentWeight ? profileData.currentWeight + 'kg' : '') }}</text>
          <uni-icons type="forward" size="16" color="#999"></uni-icons>
        </view>
        <view class="profile-item" @click="$refs.targetWeightKeyboard.open(parseFloat(weightGoal.targetWeight) || '')">
          <text class="label">目标体重</text>
          <text class="value">{{ tempValues.targetWeight || (weightGoal.targetWeight ? weightGoal.targetWeight + 'kg' : '') }}</text>
          <uni-icons type="forward" size="16" color="#999"></uni-icons>
        </view>
      </view>


      <!-- 底部保存按钮 -->
      <view class="save-button-container">
        <button class="save-button" @tap="saveProfile">保存</button>
      </view>

      <DatePicker ref="datePicker" :field="currentField" @save="handleDateSave" />

      <NumberKeyboard ref="heightKeyboard" field="height" title="请输入身高" unit="cm" :max-length="5" :range="[50, 250]"
        :decimal-digits="1" @confirm="handleNumberConfirm" @input="handleNumberInput" />

      <NumberKeyboard ref="currentWeightKeyboard" field="currentWeight" title="请输入当前体重" unit="kg" :max-length="5"
        :range="[30, 200]" :decimal-digits="1" @confirm="handleNumberConfirm" @input="handleNumberInput" />

      <NumberKeyboard ref="initialWeightKeyboard" field="initialWeight" title="请输入初始体重" unit="kg" :max-length="5"
        :range="[30, 200]" :decimal-digits="1" @confirm="handleNumberConfirm" @input="handleNumberInput" />

      <NumberKeyboard ref="targetWeightKeyboard" field="targetWeight" title="请输入目标体重" unit="kg" :max-length="5"
        :range="[30, 200]" :decimal-digits="1" @confirm="handleNumberConfirm" @input="handleNumberInput" />

      <!-- 自定义数字键盘弹窗 -->
    </view>
  </view>
</template>

<script>
import DatePicker from '@/components/DatePicker.vue';
import NumberKeyboard from '@/components/NumberKeyboard.vue'

const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  components: {
    DatePicker,
    NumberKeyboard
  },
  data() {
    return {
      weightGoal: {
        startWeight: '',
        currentWeight: '',
        targetWeight: ''
      },
      profileData: {
        username: 'Name',
        gender: '男',
        birthdate: '2005-10-26',
        height: '170',
        currentWeight: '',
        initialWeight: '',
        initialDate: '2024-08-29'
      },
      showDatePicker: false,
      showNumberPad: false,
      editFieldName: '',
      editTitle: '',
      inputValue: '',
      minValue: 0,
      maxValue: 0,
      dateRange: [[], [], []], // 年、月、日范围
      dateValue: [0, 0, 0], // 当前选中的年、月、日索引
      numberPadAnimation: '',
      currentField: '',
      tempValues: {
        height: null,
        currentWeight: null,
        initialWeight: null,
        targetWeight: null
      }
    };
  },
  onLoad() {
    this.initDateRange();
    this.fetchUserInfo();
  },
  methods: {
    async fetchUserInfo() {
      try {
        // 获取用户基本信息
        const userResponse = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/user/info',
            method: 'GET',
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        });

        // 获取体重目标信息
        const weightResponse = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/weight/goal/getGoal',
            method: 'GET',
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        });

        if (userResponse.statusCode === 200) {
          const userInfo = userResponse.data;
          this.profileData.username = userInfo.userName || '未命名用户';
          this.profileData.gender = userInfo.gender || '未知';
          this.profileData.birthdate = userInfo.birthdate || '2000-01-01';
          this.profileData.height = userInfo.height || '';
        }

        if (weightResponse.statusCode === 200) {
          this.weightGoal = weightResponse.data;
          this.profileData.initialWeight = this.weightGoal.startWeight || 0;
          this.profileData.currentWeight = this.weightGoal.currentWeight || 0;
        }
      } catch (err) {
        console.error('获取用户信息失败：', err);
      }
    },
    async updateUserInfo() {
      try {
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/user/update',
            method: 'POST',
            data: {
              userName: this.profileData.username,
              birthdate: this.profileData.birthdate,
              gender: this.profileData.gender,
              height: this.profileData.height,
            },
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')  
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

      
        await this.fetchUserInfo(); 
      } catch (err) {
        console.error('更新失败：', err)
      }
    },
    async updateUserWeightGoal(){
      try {

        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/weight/goal/save',
            method: 'POST',
            data: {
              startWeight: this.profileData.initialWeight,
              currentWeight: this.profileData.currentWeight,
              targetWeight: this.weightGoal.targetWeight
            },
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

      } catch (err) {
        console.error('更新失败：', err)
      }
    },
    // 初始化日期范围
    initDateRange() {
      const years = [];
      const months = [];
      const days = [];
      const now = new Date();
      const currentYear = now.getFullYear();
      for (let i = currentYear - 100; i <= currentYear; i++) years.push(i);
      for (let i = 1; i <= 12; i++) months.push(i < 10 ? `0${i}` : `${i}`);
      for (let i = 1; i <= 31; i++) days.push(i < 10 ? `0${i}` : `${i}`);
      this.dateRange = [years, months, days];
    },
    editField(field) {
      this.editFieldName = field;
      switch (field) {
        case 'username':
          uni.showModal({
            title: '修改用户名',
            editable: true,
            placeholderText: '请输入用户名',
            content: this.profileData.username,
            success: (res) => {
              if (res.confirm && res.content) {
                this.profileData.username = res.content;
              }
            }
          });
          break;
        case 'gender':
          uni.showActionSheet({
            itemList: ['男', '女'],
            success: (res) => {
              if (res.tapIndex !== undefined) {
                this.profileData.gender = res.tapIndex === 0 ? '男' : '女';
              }
            }
          });
          break;
      }
    },
    onDateChange(e) {
      this.dateValue = e.detail.value;
    },
    saveDate() {
      const [yearIdx, monthIdx, dayIdx] = this.dateValue;
      const date = `${this.dateRange[0][yearIdx]}-${this.dateRange[1][monthIdx]}-${this.dateRange[2][dayIdx]}`;
      this.profileData[this.editFieldName] = date;
      this.showDatePicker = false;
    },
    closeDatePicker() {
      this.showDatePicker = false;
    },
    appendNumber(number) {
      if (number === '.' && this.inputValue.includes('.')) return;
      this.inputValue += number;
    },
    deleteNumber() {
      this.inputValue = this.inputValue.slice(0, -1);
    },
    saveNumber() {
      let value = parseFloat(this.inputValue);
      if (isNaN(value) || this.inputValue === '') {
        uni.showToast({ title: '请输入有效数字', icon: 'none' });
        return;
      }
      value = Math.max(this.minValue, Math.min(this.maxValue, value));
      if (this.editFieldName === 'height') {
        this.profileData.height = `${value} 厘米`;
      } else if (this.editFieldName === 'currentWeight' || this.editFieldName === 'initialWeight') {
        this.profileData[this.editFieldName] = `${value} 公斤`;
      }
      this.animateNumberPad(false);
      setTimeout(() => { this.showNumberPad = false; }, 300);
    },
    closeNumberPad() {
      this.animateNumberPad(false);
      setTimeout(() => { this.showNumberPad = false; }, 300);
    },
    animateNumberPad(show) {
      this.numberPadAnimation = show ? 'slide-in' : 'slide-out';
    },
    saveProfile() {
      uni.showToast({ title: '保存成功', icon: 'success', duration: 2000 });
      this.updateUserInfo();
      this.updateUserWeightGoal();
    },
    openPicker(field) {
      this.currentField = field
      this.$refs.datePicker.showPicker = true
    },
    handleDateSave({ field, date }) {
      if (field === 'birth') {
        this.profileData.birthdate = date
      } else if (field === 'initial') {
        this.profileData.initialDate = date
      }
    },
    handleNumberInput({ field, value }) {
      if (field === 'height') {
        this.tempValues.height = value ? value + 'cm' : ''
      } else if (field === 'currentWeight' || field === 'initialWeight' || field === 'targetWeight') {
        this.tempValues[field] = value ? value + 'kg' : ''
      }
    },
    handleNumberConfirm({ field, value }) {
      if (field === 'height') {
        this.profileData.height = value;
        this.tempValues.height = null;
      } else if (field === 'currentWeight') {
        this.profileData.currentWeight = value;
        this.tempValues.currentWeight = null;
      } else if (field === 'initialWeight') {
        this.profileData.initialWeight = value;
        this.tempValues.initialWeight = null;
      } else if (field === 'targetWeight') {
        this.weightGoal.targetWeight = value;
        this.tempValues.targetWeight = null;
      }
    }
  }
};
</script>

<style scoped>
.page-wrapper {
  min-height: 100vh;
  background: #f5f5f5;
}

.profile-container {
  padding: 20rpx;
  background: #f5f5f5;
}

.profile-block {
  background: #fff;
  border-radius: 16rpx;
  margin-bottom: 20rpx;
}

.profile-item {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  padding: 30rpx;
  border-bottom: 1rpx solid #eee;
}

.label {
  width: 180rpx;
  font-size: 32rpx;
  color: #333;
}

.value {
  flex: 1;
  font-size: 32rpx;
  color: #666;
  text-align: left;
  padding-right: 20rpx;
}

.save-button-container {
  padding: 20rpx;
  position: fixed;
  bottom: 20rpx;
  left: 20rpx;
  right: 20rpx;
}

.save-button {
  background-color: #4cd964;
  color: #fff;
  font-size: 32rpx;
  border-radius: 40rpx;
}

/* 弹窗样式 */
.modal {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: #fff;
  border-top-left-radius: 20rpx;
  border-top-right-radius: 20rpx;
  box-shadow: 0 -2rpx 10rpx rgba(0, 0, 0, 0.1);
  z-index: 999;
  padding: 20rpx;
}

.modal-content {
  padding: 20rpx;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 20rpx;
  border-bottom: 1rpx solid #eee;
}

.modal-input {
  padding: 20rpx;
  text-align: center;
  font-size: 36rpx;
  color: #333;
}

.picker-view {
  padding: 40rpx;
  text-align: center;
  font-size: 32rpx;
}

.number-pad {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.number-row {
  display: flex;
  justify-content: space-around;
  width: 100%;
  margin-bottom: 20rpx;
}

.number-btn {
  width: 150rpx;
  height: 100rpx;
  font-size: 32rpx;
  border: 1rpx solid #ccc;
  border-radius: 12rpx;
  background: #f5f5f5;
  display: flex;
  justify-content: center;
  align-items: center;
}

.save-btn {
  width: 100%;
  height: 80rpx;
  background-color: #4cd964;
  color: #fff;
  font-size: 32rpx;
  border-radius: 40rpx;
  border: none;
  margin-top: 20rpx;
}

/* 动画 */
@keyframes slide-in {
  from {
    transform: translateY(100%);
  }

  to {
    transform: translateY(0);
  }
}

@keyframes slide-out {
  from {
    transform: translateY(0);
  }

  to {
    transform: translateY(100%);
  }
}

.modal[animation="slide-in"] {
  animation: slide-in 0.3s ease-in-out forwards;
}

.modal[animation="slide-out"] {
  animation: slide-out 0.3s ease-in-out forwards;
}
</style>