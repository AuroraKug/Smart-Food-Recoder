<template>
  <view class="profile-container">
    <view class="profile-block">
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
      <view class="profile-item" @tap="editField('height')">
        <text class="label">身高</text>
        <text class="value">{{ profileData.height }}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>
      <view class="profile-item" @tap="editField('currentWeight')">
        <text class="label">当前体重</text>
        <text class="value">{{ profileData.currentWeight }}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>
    </view>

    <view class="profile-block">
      <view class="profile-item" @tap="editField('initialWeight')">
        <text class="label">初始体重</text>
        <text class="value">{{ profileData.initialWeight }}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>
      <view class="profile-item" @click="openPicker('initial')">
        <text class="label">初始日期</text>
        <text class="value">{{ profileData.initialDate }}</text>
        <uni-icons type="forward" size="16" color="#999"></uni-icons>
      </view>
    </view>

    <!-- 底部保存按钮 -->
    <view class="save-button-container">
      <button class="save-button" @tap="saveProfile">保存</button>
    </view>

    <!-- 自定义日期选择弹窗 -->
    <!-- <view class="modal" v-if="showDatePicker">
      <view class="modal-content">
        <view class="modal-header">
          <text>{{ editTitle }}</text>
          <uni-icons type="close" size="24" color="#666" @tap="closeDatePicker"></uni-icons>
        </view>
        <picker mode="multiSelector" :range="dateRange" :value="dateValue" @change="onDateChange">
          <view class="picker-view">
            {{ dateRange[0][dateValue[0]] }}-{{ dateRange[1][dateValue[1]] }}-{{ dateRange[2][dateValue[2]] }}
          </view>
        </picker>
        <button class="save-btn" @tap="saveDate">保存</button>
      </view>
    </view> -->
    <DatePicker
      ref="datePicker"
      :field="currentField"
      @save="handleDateSave"
    />

    

    <!-- 自定义数字键盘弹窗 -->
    <view class="modal" v-if="showNumberPad" animation="{{ numberPadAnimation }}">
      <view class="modal-content">
        <view class="modal-header">
          <text>{{ editTitle }}</text>
          <uni-icons type="close" size="24" color="#666" @tap="closeNumberPad"></uni-icons>
        </view>
        <view class="modal-input">
          <text>{{ inputValue }}</text>
        </view>
        <view class="number-pad">
          <view class="number-row">
            <button class="number-btn" @tap="appendNumber('1')">1</button>
            <button class="number-btn" @tap="appendNumber('2')">2</button>
            <button class="number-btn" @tap="appendNumber('3')">3</button>
            <button class="number-btn" @tap="appendNumber('4')">4</button>
          </view>
          <view class="number-row">
            <button class="number-btn" @tap="appendNumber('5')">5</button>
            <button class="number-btn" @tap="appendNumber('6')">6</button>
            <button class="number-btn" @tap="appendNumber('7')">7</button>
            <button class="number-btn" @tap="appendNumber('8')">8</button>
          </view>
          <view class="number-row">
            <button class="number-btn" @tap="appendNumber('9')">9</button>
            <button class="number-btn" @tap="appendNumber('0')">0</button>
            <button class="number-btn" @tap="appendNumber('.')">.</button>
            <button class="number-btn" @tap="deleteNumber">←</button>
          </view>
        </view>
        <button class="save-btn" @tap="saveNumber">保存</button>
      </view>
    </view>
  </view>
</template>

<script>
import DatePicker from './components/DatePicker.vue';
export default {
  components: {
    DatePicker
  },
  data() {
    return {
      profileData: {
        gender: '男',
        birthdate: '2005-10-26',
        height: '180 厘米',
        currentWeight: '66.4 公斤',
        initialWeight: '70.6 公斤',
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
      currentField: ''
    };
  },
  onLoad() {
    this.initDateRange();
  },
  methods: {
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
        case 'birthdate':
        case 'initialDate':
          this.editTitle = field === 'birthdate' ? '编辑生日' : '编辑初始日期';
          const dateParts = this.profileData[field].split('-');
          this.dateValue = [
            this.dateRange[0].indexOf(parseInt(dateParts[0])),
            this.dateRange[1].indexOf(dateParts[1]),
            this.dateRange[2].indexOf(dateParts[2])
          ];
          this.showDatePicker = true;
          break;
        case 'height':
          this.editTitle = '编辑身高';
          this.inputValue = this.profileData.height.replace(' 厘米', '');
          this.minValue = 100;
          this.maxValue = 250;
          this.showNumberPad = true;
          this.animateNumberPad(true);
          break;
        case 'currentWeight':
        case 'initialWeight':
          this.editTitle = field === 'currentWeight' ? '编辑当前体重' : '编辑初始体重';
          this.inputValue = this.profileData[field].replace(' 公斤', '');
          this.minValue = 25;
          this.maxValue = 200;
          this.showNumberPad = true;
          this.animateNumberPad(true);
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
      console.log('保存数据:', this.profileData);
      uni.showToast({ title: '保存成功', icon: 'success', duration: 2000 });
    },
    openPicker(field) {
      this.currentField = field
      this.$refs.datePicker.showPicker = true
    },
    handleDateSave({field, date}) {
      if (field === 'birth') {
        this.profileData.birthdate = date
      } else if (field === 'initial') {
        this.profileData.initialDate = date
      }
    }
  }
};
</script>

<style scoped>
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
  justify-content: space-between;
  padding: 30rpx;
  border-bottom: 1rpx solid #eee;
}

.label {
  font-size: 32rpx;
  color: #333;
}

.value {
  font-size: 32rpx;
  color: #666;
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