<template>
  <view class="photo-section">
    <navigator url="/pages/camera/camera" class="photo-btn">
      <uni-icons type="camera" size="24" color="#fff"></uni-icons>
      <text>拍照记录饮食</text>
    </navigator>

    <view class="quick-actions">
      <view class="action-item" @click="$refs.weightKeyboard.open(currentValue)">
        <uni-icons type="scale" size="20"></uni-icons>
        <text>记录体重</text>
      </view>
      <!-- <view class="action-item" @click="goToWater">
        <uni-icons type="water" size="20"></uni-icons>
        <text>喝水打卡</text>
      </view> -->
    </view>

    <!-- <uni-popup ref="weightPopup" type="bottom" @change="popupChange">
      <view class="custom-weight-popup">
        <view class="popup-header">
          <text class="title">记录体重</text>
          <uni-icons type="close" size="24" color="#999" @click="closePopup"></uni-icons>
        </view>

        <view class="weight-display">
          <text class="value">{{ currentValue }}</text>
          <text class="unit">kg</text>
        </view>

        <view class="number-pad">
          <view
            class="number-btn"
            v-for="num in [1, 2, 3, 4, 5, 6, 7, 8, 9, '.', 0]"
            :key="num"
            @click="inputNum(num)"
          >
            {{ num }}
          </view>
          <view class="number-btn delete" @click="deleteNum">
            <uni-icons type="back" size="24" color="#666"></uni-icons>
          </view>
        </view>

        <button class="confirm-btn" :class="{ disabled: !isValid }" @click="confirmWeight">
          确认记录
        </button>
      </view>
    </uni-popup> -->
    <NumberKeyboard ref="weightKeyboard" field="currentValue" title="请输入当前体重" unit="kg" :max-length="5"
      :range="[30, 200]" :decimal-digits="1" @confirm="handleNumberConfirm" />

  </view>
</template>

<script>
import NumberKeyboard from '@/components/NumberKeyboard.vue'
export default {
  components: {
    NumberKeyboard
  },
  props: {
    lastWeight: {
      type: Number,
      default: null
    },
    initialWeightData: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      currentValue: ''
    }
  },
  computed: {
    isValid() {
      return /^\d+\.?\d*$/.test(this.currentValue) &&
        parseFloat(this.currentValue) > 20 &&
        parseFloat(this.currentValue) < 200
    }
  },
  methods: {
    goToCamera() {
      uni.navigateTo({
        url: '/pages/camera/camera'
      })
    },
    goToWater() {
      uni.navigateTo({
        url: '/pages/water/water'
      })
    },
    showWeightInput() {
      this.$nextTick(() => {
        if (this.$refs.weightPopup) {
          this.$refs.weightPopup.open()
        }
      })
    },
    popupChange(e) {
      this.$emit('popup-change', e.show)
    },
    closePopup() {
      this.$refs.weightPopup.close()
    },
    inputNum(num) {
      if (num === '.' && this.currentValue.includes('.')) return
      if (this.currentValue.length >= 5) return

      this.currentValue += num.toString()
      if (num === '.' && !this.currentValue.includes('.')) {
        this.currentValue += '0'
      }
    },
    deleteNum() {
      this.currentValue = this.currentValue.slice(0, -1);
    },
    confirmWeight() {
      if (!this.isValid) {
        uni.showToast({ title: '请输入有效体重', icon: 'none' })
        return
      }

      const weight = parseFloat(this.currentValue)
      this.$emit('confirm-weight', weight)
      this.closePopup()

      uni.showToast({ title: `成功记录: ${weight}kg`, icon: 'success' })
    },
    handleNumberConfirm({ field, value }) {
      if (field === 'currentValue') {
        this.currentValue = value + 'kg'
      }
    }
  }
}
</script>

<style scoped>
.photo-section {
  width: 90%;
  margin: 0 auto;
}

.photo-btn {
  background: linear-gradient(to right, #4cd964, #2fd178);
  color: white;
  border: none;
  border-radius: 50rpx;
  height: 90rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 30rpx 0;
}

.quick-actions {
  display: flex;
  justify-content: space-around;
  margin: 30rpx 0;
}

.action-item {
  background: white;
  width: 45%;
  text-align: center;
  padding: 20rpx 0;
  border-radius: 12rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
}

.custom-weight-popup {
  background: #fff;
  border-radius: 24rpx 24rpx 0 0;
  padding: 20rpx;
  height: auto;
  max-height: 60vh;
  box-sizing: border-box;
}

.popup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
}

.weight-display {
  display: flex;
  justify-content: center;
  align-items: flex-end;
  margin: 20rpx 0;
}

.value {
  font-size: 60rpx;
  font-weight: bold;
  color: #333;
}

.unit {
  font-size: 28rpx;
  color: #999;
  margin-left: 10rpx;
  padding-bottom: 8rpx;
}

.number-pad {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16rpx;
  margin-bottom: 20rpx;
}

.number-btn {
  height: 80rpx;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #f5f5f5;
  border-radius: 12rpx;
  font-size: 36rpx;
}

.confirm-btn {
  background: #4cd964;
  color: white;
  border: none;
  border-radius: 50rpx;
  height: 80rpx;
  font-size: 28rpx;
  width: 100%;
}

.confirm-btn.disabled {
  background: #cccccc;
  opacity: 0.7;
}
</style>