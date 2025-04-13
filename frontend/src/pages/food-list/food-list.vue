<template>
  <view class="container">
    <view class="food-list-container">
      <view class="food-list-title">{{ searchKeyword ? '搜索结果' : '食物列表' }}</view>
      <view class="food-item" v-for="(food, index) in foodList" :key="index">
        <image :src="food.image" class="food-image" mode="widthFix"></image>
        <view class="food-info">
          <text class="food-name">{{ food.name }}</text>
          <text class="food-calories">{{ food.calories }}卡/100克</text>
        </view>
        <view class="add-button" @click="showRecordPopup(food)">
          <uni-icons type="plus" size="24" color="#4cd964"></uni-icons>
        </view>
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
            <text class="food-calories">{{ selectedFood ? selectedFood.calories : 0 }}卡/100克</text>
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
      foodList: [],
      searchKeyword: '',
      selectedFood: null,
      weight: '100',
      totalCalories: 0
    }
  },
  onLoad(options) {
    // 获取URL参数中的搜索关键词
    if (options.keyword) {
      this.searchKeyword = decodeURIComponent(options.keyword)
      this.handleSearch(this.searchKeyword)
    }
  },
  methods: {
    async handleSearch(keyword) {
      try {
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/food/search',
            method: 'GET',
            data: {
              keyword: keyword
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
          this.foodList = response.data
        }
      } catch (err) {
        console.error('搜索失败：', err)
        uni.showToast({
          title: '搜索失败',
          icon: 'none'
        })
      }
    },
    showRecordPopup(food) {
      this.selectedFood = food
      this.weight = '100'
      this.calculateCalories()
      this.$refs.recordPopup.open()
      // 使用微信小程序的方式禁止页面滚动
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
    calculateCalories() {
      if (this.selectedFood && this.weight) {
        this.totalCalories = Math.round(this.selectedFood.calories * parseFloat(this.weight) / 100)
      } else {
        this.totalCalories = 0
      }
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
    async recordFood() {
      if (!this.selectedFood || !this.weight) {
        uni.showToast({
          title: '请填写完整信息',
          icon: 'none'
        })
        return
      }

      console.log(this.selectedFood);
      console.log(this.weight);
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
              caloriesPer100g: parseFloat(this.selectedFood.calories),
              weight: parseFloat(this.weight),
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
  padding: 20rpx;
}

.food-list-container {
  padding: 20rpx;
  min-height: 100vh;
}

.food-list-title {
  font-size: 40rpx;
  font-weight: bold;
  color: #333333;
  text-align: center;
  margin-bottom: 30rpx;
}

.food-item {
  display: flex;
  align-items: center;
  padding: 20rpx;
  background-color: #ffffff;
  border-radius: 12rpx;
  margin: 0 auto 20rpx;
  box-shadow: 0rpx 2rpx 8rpx 2rpx #cccccc;
  overflow: hidden;
  position: relative;
}

.food-image {
  width: 120rpx;
  height: 120rpx;
  border-radius: 12rpx;
  margin-right: 24rpx;
  background-color: #f0f0f0;
}

.food-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.food-name {
  font-size: 34rpx;
  color: #333333;
  font-weight: 500;
  line-height: 1.2;
  margin-bottom: 8rpx;
}

.food-calories {
  font-size: 28rpx;
  color: #666666;
  line-height: 1.2;
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

.unit {
  font-size: 28rpx;
  color: #666666;
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