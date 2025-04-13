<template>
  <view class="container">
    <image :src="capturedImagePath" class="background" mode="widthFix"></image>
    <view class="whiteboard"
      :style="{ height: whiteboardHeight + 'px', transition: 'height 0.3s cubic-bezier(0.42, 0, 0.58, 1)' }"
      @touchstart="handleTouchStart" @touchmove.stop="handleTouchMove" @touchend="handleTouchEnd">
      <view class="drag-handle"></view>

      <!-- 顶部信息区 -->
      <view class="top-section" v-if="!mainResult || (mainResult && mainResult.name !== '非菜')">
        <view class="food-header">
          <view class="icon-wrapper">
            <image :src="capturedImagePath" class="food-icon" mode="aspectFit" />
          </view>

          <view class="food-info">
            <text class="food-name" v-if="!mainResult">识别中...</text>
            <text class="food-name" v-else>{{ mainResult.name }}</text>

            <view class="progress-container" v-if="mainResult && mainResult.name !== '非菜'">
              <uni-icons type="eye" size="18" color="#999"></uni-icons>
              <view class="progress-bar">
                <view class="progress-fill" :style="{
                  width: progress + '%',
                  background: getProgressColor(progress)
                }"></view>
              </view>
              <text class="progress-value">{{ progress }}%</text>
            </view>
            <view class="add-button" v-if="mainResult && mainResult.name !== '非菜'" @click="showRecordPopup(mainResult)">
              <uni-icons type="plus" size="24" color="#4cd964"></uni-icons>
            </view>
          </view>
        </view>
      </view>

      <!-- 营养卡片 -->
      <text class="title-font" v-if="mainResult && mainResult.name !== '非菜'">营养</text>
      <view class="nutrition-card" v-if="mainResult && mainResult.name !== '非菜'"> 
        <view class="nutrition-row">
          <text class="label">热量</text>
          <view class="value-wrapper">
            <text class="value">{{ mainResult ? mainResult.calorie : '--' }}</text>
            <text class="unit">卡/100克</text>
          </view>
        </view>
      </view>

      <!-- 其他可能结果
      <text class="title-font" v-if="otherResults.length > 0 && mainResult && mainResult.name !== '非菜'">其他可能结果</text>
      <view class="other-results" v-if="otherResults.length > 0 && mainResult && mainResult.name !== '非菜'">
        <view class="other-result-item" v-for="(item, index) in otherResults" :key="index">
          <text class="other-food-name">{{ item.name }}</text>
          <text class="other-food-calories">{{ item.calorie }}卡/100克</text>
          <text class="other-probability">{{ (item.probability * 100).toFixed(2) }}%</text>
        </view>
      </view> -->
      <text class="title-font" v-if="otherResults.length > 0 && mainResult && mainResult.name !== '非菜'">其他可能结果</text>
      <view class="other-results" v-if="otherResults.length > 0 && mainResult && mainResult.name !== '非菜'">
        <view class="other-result-item" v-for="(item, index) in otherResults" :key="index">
          <view class="result-row">
            <text class="other-food-name">{{ item.name }}</text>
            <text class="other-food-calories">{{ item.calorie }}卡/100克</text>
            <text class="other-probability">{{ (item.probability * 100).toFixed(2) }}%</text>
            <view class="add-button" @click="showRecordPopup(item)">
              <uni-icons type="plus" size="24" color="#4cd964"></uni-icons>
            </view>
          </view>
        </view>
      </view>


      <!-- 非菜提示 -->
      <view class="non-food-warning" v-if="mainResult && mainResult.name === '非菜'">
        <text class="warning-text">⚠️ 识别结果可能不是食物</text>
        <text class="warning-desc">请尝试重新拍摄或选择其他食物</text>
      </view>

      <!-- 操作按钮 -->
      <view class="button-container">
        <button class="action-button" @click="retakePhoto">重新拍摄</button>
        <button class="action-button" @click="confirmSelection" v-if="mainResult && mainResult.name !== '非菜'">确认选择</button>
      </view>
    </view>

    <!-- 记录弹窗 -->
    <uni-popup ref="recordPopup" type="center">
      <view class="record-popup">
        <view class="popup-header">
          <text class="popup-title">记录食物</text>
          <uni-icons type="close" size="24" color="#999" @click="closeRecordPopup"></uni-icons>
        </view>
        <view class="popup-content">
          <view class="food-info">
            <text class="food-name">{{ selectedFood ? selectedFood.name : '' }}</text>
            <text class="food-calories">{{ selectedFood ? selectedFood.calorie : 0 }}卡/100克</text>
          </view>
          <view class="weight-input">
            <text class="label">食用重量</text>
            <input 
              type="digit" 
              v-model="weight" 
              class="input" 
              placeholder="请输入重量"
              @input="handleWeightInput"
              maxlength="6"
            />
            <text class="unit">克</text>
          </view>
          <view class="total-calories">
            <text class="label">总卡路里</text>
            <text class="value">{{ totalCalories }}卡</text>
          </view>
        </view>
        <view class="popup-footer">
          <button class="cancel-button" @click="closeRecordPopup">取消</button>
          <button class="confirm-button" @click="recordFood">确认</button>
        </view>
      </view>
    </uni-popup>
  </view>
</template>

<script>
const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'

export default {
  data() {
    return {
      startY: 0,
      currentY: 0,
      whiteboardHeight: 0,
      minHeight: 0,
      maxHeight: 0,
      capturedImagePath: '',
      progress: 0,
      mainResult: null,
      otherResults: [],
      hasLowProbability: false,
      selectedFood: null,
      weight: '100',
      totalCalories: 0
    }
  },
  onLoad(options) {
    this.capturedImagePath = options.imagePath ? decodeURIComponent(options.imagePath) : ''
    const systemInfo = uni.getSystemInfoSync()
    const screenHeight = systemInfo.windowHeight
    this.minHeight = Math.floor(screenHeight * 0.3)
    this.maxHeight = Math.floor(screenHeight * 0.85)
    this.whiteboardHeight = this.minHeight
    this.identifyFood()
  },
  methods: {
    handleTouchStart(event) {
      this.startY = event.touches[0].pageY
      this.currentY = this.startY
    },
    handleTouchMove(event) {
      const moveY = event.touches[0].pageY
      const diffY = this.startY - moveY
      this.whiteboardHeight = Math.max(this.minHeight, Math.min(this.maxHeight, this.whiteboardHeight + diffY))
      this.currentY = moveY
    },
    handleTouchEnd() {
      const lastDirection = this.currentY - this.startY
      if (lastDirection < 0) {
        this.whiteboardHeight = this.maxHeight
      } else {
        this.whiteboardHeight = this.minHeight
      }
    },
    getProgressColor(percent) {
      const hue = percent * 120 / 100
      return `hsl(${hue}, 100%, 50%)`
    },
    async identifyFood() {
      try {
        const response = await new Promise((resolve, reject) => {
          uni.uploadFile({
            url: BASE_URL + '/api/photo',
            filePath: this.capturedImagePath,
            name: 'file',
            header: {
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

        if (response.statusCode === 200) {
          const data = JSON.parse(response.data)
          if (data.result && data.result.length > 0) {
            // 设置主要结果
            this.mainResult = data.result[0]
            this.progress = (this.mainResult.probability * 100).toFixed(2)
            
            // 过滤其他结果（概率大于10%）
            this.otherResults = data.result.slice(1).filter(item => 
              parseFloat(item.probability) > 0.05
            )
            
            // 检查是否有低概率结果
            this.hasLowProbability = data.result.some(item => 
              parseFloat(item.probability) <= 0.05
            )
          }
        }
      } catch (err) {
        console.error('识别失败：', err)
        uni.showToast({
          title: '识别失败',
          icon: 'none'
        })
      }
    },
    retakePhoto() {
      uni.navigateBack()
    },
    confirmSelection() {
      if (this.mainResult) {
        this.$emit('select', this.mainResult)
        uni.navigateBack()
      }
    },
    showRecordPopup(food) {
      this.selectedFood = food
      this.weight = '100'
      this.calculateCalories()
      this.$refs.recordPopup.open()
      uni.pageScrollTo({
        scrollTop: 0,
        duration: 0
      })
    },
    closeRecordPopup() {
      this.$refs.recordPopup.close()
      this.selectedFood = null
      this.weight = '100'
      this.totalCalories = 0
    },
    handleWeightInput(e) {
      // 只允许输入数字和小数点
      let value = e.detail.value.replace(/[^\d.]/g, '')
      // 只允许一个小数点
      if ((value.match(/\./g) || []).length > 1) {
        value = value.slice(0, -1)
      }
      // 小数点后最多两位
      if (value.includes('.')) {
        const parts = value.split('.')
        if (parts[1].length > 2) {
          value = parts[0] + '.' + parts[1].slice(0, 2)
        }
      }
      this.weight = value
      this.calculateCalories()
    },
    calculateCalories() {
      if (this.selectedFood && this.weight) {
        this.totalCalories = Math.round(this.selectedFood.calorie * parseFloat(this.weight) / 100)
      } else {
        this.totalCalories = 0
      }
    },
    async recordFood() {
      if (!this.selectedFood || !this.weight) {
        uni.showToast({
          title: '请填写完整信息',
          icon: 'none'
        })
        return
      }

      try {
        const now = new Date()
        const recordTime = now.getFullYear() + '-' +
          String(now.getMonth() + 1).padStart(2, '0') + '-' +
          String(now.getDate()).padStart(2, '0') + ' ' +
          String(now.getHours()).padStart(2, '0') + ':' +
          String(now.getMinutes()).padStart(2, '0')

        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/food/record',
            method: 'POST',
            data: {
              foodName: this.selectedFood.name,
              caloriesPer100g: parseFloat(this.selectedFood.calorie),
              weight: parseFloat(this.weight),
              recordTime: recordTime,
              totalCalories: this.totalCalories
            },
            header: {
              'Content-Type': 'application/json',
              'Authorization': 'Bearer ' + uni.getStorageSync('token')
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          })
        })

        if (response.statusCode === 200) {
          uni.showToast({
            title: '记录成功',
            icon: 'success'
          })
          this.closeRecordPopup()
        }
      } catch (err) {
        console.error('记录失败：', err)
        uni.showToast({
          title: '记录失败',
          icon: 'none'
        })
      }
    }
  }
}
</script>

<style scoped>
.container {
  position: relative;
  height: 100vh;
  overflow: hidden;
}

.background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-repeat: no-repeat;
  z-index: 1;
  touch-action: none;
}

.whiteboard {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background-color: #f2f2f2;
  border-top-left-radius: 20rpx;
  border-top-right-radius: 20rpx;
  z-index: 2;
  box-shadow: 0 -2rpx 10rpx rgba(0, 0, 0, 0.1);
  touch-action: none;
  padding: 30rpx;
  box-sizing: border-box;
}

.drag-handle {
  width: 80rpx;
  height: 8rpx;
  background-color: #333;
  border-radius: 4rpx;
  position: absolute;
  top: 20rpx;
  left: 50%;
  transform: translateX(-50%);
}

.top-section {
  height: 200rpx;
  margin-bottom: 40rpx;
}

.food-header {
  display: flex;
  align-items: center;
  height: 100%;
}

.icon-wrapper {
  width: 120rpx;
  height: 120rpx;
  background: #FFF4E6;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 30rpx;
}

.food-icon {
  width: 80rpx;
  height: 80rpx;
}

.food-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.food-name {
  font-size: 36rpx;
  font-weight: 500;
  color: #333;
  margin-bottom: 12rpx;
}

.progress-container {
  display: flex;
  align-items: center;
  margin-top: 15rpx;
}

.progress-bar {
  flex: 1;
  height: 8rpx;
  background: #b3b3b3;
  border-radius: 4rpx;
  margin: 0 15rpx;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  border-radius: 4rpx;
  transition: all 0.3s ease;
}

.progress-value {
  font-size: 24rpx;
  color: #666;
}

.title-font {
  font-size: 32rpx;
  color: #999999;
  padding: 20rpx 0;
}

.nutrition-card {
  background: #F8F9FA;
  border-radius: 16rpx;
  padding: 30rpx;
  margin-bottom: 20rpx;
}

.nutrition-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20rpx 0;
}

.label {
  font-size: 30rpx;
  color: #666;
}

.value-wrapper {
  display: flex;
  align-items: baseline;
}

.value {
  font-size: 36rpx;
  color: #4CAF50;
  font-weight: bold;
  margin-right: 10rpx;
}

.unit {
  font-size: 24rpx;
  color: #999;
}

.other-results {
  background: #F8F9FA;
  border-radius: 16rpx;
  padding: 20rpx;
  margin-bottom: 20rpx;
}

.other-result-item:last-child {
  border-bottom: none;
}

.result-row {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 1rpx solid #eeeeee;
}

.other-food-name {
  flex: 2;
  font-size: 28rpx;
  color: #333333;
  text-align: left;
}

.other-food-calories {
  flex: 3;
  font-size: 24rpx;
  color: #666666;
  text-align: center;
}

.other-probability {
  flex: 2;
  font-size: 24rpx;
  color: #999999;
  text-align: right;
}

.add-button {
  width: 60rpx;
  height: 60rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #ffffff;
  border-radius: 50%;
  margin-left: 20rpx;
}

.non-food-warning {
  background-color: #fff3f3;
  border-radius: 12rpx;
  padding: 30rpx;
  margin: 40rpx 0;
  text-align: center;
}

.warning-text {
  font-size: 32rpx;
  color: #ff4d4f;
  font-weight: bold;
  margin-bottom: 10rpx;
  display: block;
}

.warning-desc {
  font-size: 28rpx;
  color: #666;
  display: block;
  margin-top: 10rpx;
}

.button-container {
  display: flex;
  justify-content: space-around;
  margin-top: 40rpx;
}

.action-button {
  width: 45%;
  height: 80rpx;
  line-height: 80rpx;
  text-align: center;
  border-radius: 40rpx;
  font-size: 32rpx;
  color: #ffffff;
  background-color: #4cd964;
}

.action-button:active {
  opacity: 0.8;
}

/* 弹窗样式 */
.record-popup {
  width: 600rpx;
  background-color: #ffffff;
  border-radius: 20rpx;
  padding: 30rpx;
}

.popup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30rpx;
}

.popup-title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333333;
}

.popup-content {
  margin-bottom: 30rpx;
}

.weight-input {
  display: flex;
  align-items: center;
  margin: 30rpx 0;
}

.input {
  flex: 1;
  height: 80rpx;
  background-color: #f8f8f8;
  border-radius: 8rpx;
  padding: 0 20rpx;
  margin: 0 20rpx;
  font-size: 28rpx;
}

.total-calories {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20rpx;
}

.total-calories .label {
  font-size: 28rpx;
  color: #666666;
}

.total-calories .value {
  font-size: 32rpx;
  color: #4cd964;
  font-weight: bold;
}

.popup-footer {
  display: flex;
  justify-content: space-between;
  margin-top: 30rpx;
}

.cancel-button {
  width: 45%;
  height: 80rpx;
  line-height: 80rpx;
  text-align: center;
  border-radius: 8rpx;
  font-size: 32rpx;
  color: #666666;
  background-color: #f8f8f8;
}

.confirm-button {
  width: 45%;
  height: 80rpx;
  line-height: 80rpx;
  text-align: center;
  border-radius: 8rpx;
  font-size: 32rpx;
  color: #ffffff;
  background-color: #4cd964;
}
</style>