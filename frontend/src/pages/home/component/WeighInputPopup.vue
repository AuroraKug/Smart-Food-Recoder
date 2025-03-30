<template>
    <uni-popup ref="popup" type="bottom" @change="popupChange">
      <view class="weight-popup">
        <view class="popup-header">
          <text class="title">记录体重</text>
          <uni-icons 
            type="close" 
            size="24" 
            color="#999" 
            @click="close"
          ></uni-icons>
        </view>
        
        <view class="weight-display">
          <text class="value">{{ currentValue }}</text>
          <text class="unit">kg</text>
        </view>
        
        <view class="number-pad">
          <view 
            class="number-btn" 
            v-for="num in [1,2,3,4,5,6,7,8,9,'.',0]" 
            :key="num"
            @click="inputNum(num)"
          >
            {{ num }}
          </view>
          <view class="number-btn delete" @click="deleteNum">
            <uni-icons type="backspace" size="24" color="#666"></uni-icons>
          </view>
        </view>
        
        <button 
          class="confirm-btn" 
          :class="{ disabled: !isValid }"
          @click="confirm"
        >
          确认记录
        </button>
      </view>
    </uni-popup>
  </template>
  
  <script>
  export default {
    props: {
      lastWeight: {
        type: Number,
        default: null
      }
    },
    data() {
      return {
        currentValue: this.lastWeight ? this.lastWeight.toString() : '',
        isShow: false
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
      open() {
        this.$refs.popup.open()
      },
      close() {
        this.$refs.popup.close()
      },
      popupChange(e) {
        this.isShow = e.show
      },
      inputNum(num) {
        if (num === '.' && this.currentValue.includes('.')) return
        if (this.currentValue.length >= 5) return
        
        this.currentValue += num.toString()
        // 自动补全小数位
        if (num === '.' && !this.currentValue.includes('.')) {
          this.currentValue += '0'
        }
      },
      deleteNum() {
        this.currentValue = this.currentValue.slice(0, -1)
      },
      confirm() {
        if (!this.isValid) {
          uni.showToast({ title: '请输入有效体重', icon: 'none' })
          return
        }
        const weight = parseFloat(this.currentValue)
        this.$emit('confirm', weight)
        this.close()
        uni.showToast({ title: '记录成功' })
      }
    }
  }
  </script>
  
  <style scoped>
  .weight-popup {
    background: #fff;
    border-radius: 24rpx 24rpx 0 0;
    padding: 40rpx;
    padding-bottom: calc(40rpx + env(safe-area-inset-bottom));
  }
  
  .popup-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 40rpx;
  }
  
  .title {
    font-size: 36rpx;
    font-weight: bold;
  }
  
  .weight-display {
    display: flex;
    justify-content: center;
    align-items: flex-end;
    margin: 40rpx 0;
  }
  
  .value {
    font-size: 80rpx;
    font-weight: bold;
    color: #333;
  }
  
  .unit {
    font-size: 36rpx;
    color: #999;
    margin-left: 10rpx;
    padding-bottom: 12rpx;
  }
  
  .number-pad {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20rpx;
    margin-bottom: 40rpx;
  }
  
  .number-btn {
    height: 100rpx;
    display: flex;
    justify-content: center;
    align-items: center;
    background: #f5f5f5;
    border-radius: 12rpx;
    font-size: 40rpx;
  }
  
  .number-btn:active {
    background: #e5e5e5;
  }
  
  .delete {
    background: #f0f0f0;
  }
  
  .confirm-btn {
    background: #4CD964;
    color: white;
    border: none;
    border-radius: 50rpx;
    height: 90rpx;
    font-size: 32rpx;
  }
  
  .confirm-btn.disabled {
    background: #cccccc;
    opacity: 0.7;
  }
  </style>