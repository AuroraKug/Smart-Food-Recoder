<template>
  <view class="photo-section">
    <navigator url="/pages/camera/camera" class="photo-btn">
      <uni-icons type="camera" size="24" color="#fff"></uni-icons>
      <text>拍照记录饮食</text>
    </navigator>

    <view class="quick-actions">
      <view class="action-item" @click="openWeightKeyboard">
        <uni-icons type="scale" size="20"></uni-icons>
        <text>记录体重</text>
      </view>
    </view>

    <NumberKeyboard ref="weightKeyboard" field="currentWeight" title="请输入当前体重" unit="kg" :max-length="5"
      :range="[30, 200]" :decimal-digits="1" @confirm="handleNumberConfirm" @input="handleNumberInput" />

  </view>
</template>

<script>
import NumberKeyboard from '@/components/NumberKeyboard.vue'
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  components: {
    NumberKeyboard
  },
  data() {
    return {
      weightData: {
        startWeight: null,
        currentWeight: null,
        targetWeight: null
      },
      tempWeight: null
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
    async fetchWeightData() {
      try {
        const response = await wx.cloud.callContainer({
          path: '/api/weight/goal/getGoal',
          method: 'GET',
          header: {
            'X-WX-SERVICE': 'springboot-glwv', // ⚠️ 替换为你自己的云托管服务名
            'Authorization': 'Bearer ' + uni.getStorageSync('token'),
            'Content-Type': 'application/json'
          }
        })

        if (response.statusCode === 200) {
          this.weightData = response.data
        } else {
          throw new Error(response.data.message || '获取体重数据失败')
        }

      } catch (err) {
        console.error('获取体重数据失败：', err)
        uni.showToast({ title: '获取体重数据失败', icon: 'none' })
      }
    },

    async updateWeight(currentWeight) {
      try {
        const response = await wx.cloud.callContainer({
          path: '/api/weight/goal/save',
          method: 'POST',
          data: {
            startWeight: this.weightData.startWeight,
            currentWeight: currentWeight,
            targetWeight: this.weightData.targetWeight
          },
          header: {
            'X-WX-SERVICE': 'springboot-glwv', // ⚠️ 替换为你自己的云托管服务名
            'Authorization': 'Bearer ' + uni.getStorageSync('token'),
            'Content-Type': 'application/json'
          }
        })

        if (response.statusCode === 200) {
          uni.showToast({ title: '更新成功', icon: 'success' })
          this.weightData.currentWeight = currentWeight
          uni.$emit('weight-updated', currentWeight)
        } else {
          throw new Error(response.data.message || '更新体重失败')
        }

      } catch (err) {
        console.error('更新体重失败：', err)
        uni.showToast({ title: '更新失败', icon: 'none' })
      }
    },

    openWeightKeyboard() {
      this.fetchWeightData().then(() => {
        this.$refs.weightKeyboard.open(parseFloat(this.weightData.currentWeight) || '')
      })
    },
    handleNumberInput({ field, value }) {
      if (field === 'currentWeight') {
        this.tempWeight = value
      }
    },
    handleNumberConfirm({ field, value }) {
      if (field === 'currentWeight') {
        this.updateWeight(value)
        this.tempWeight = null
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