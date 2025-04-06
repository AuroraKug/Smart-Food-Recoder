<template>
  <view class="home-container">
    <view class="search-container">
    <!-- 搜索框 -->
    <view class="search-box">
      <uni-icons type="search" size="18" color="#999" class="search-icon"></uni-icons>
      <input 
        class="search-input" 
        placeholder="请输入搜索内容" 
        placeholder-class="placeholder-style"
        v-model="searchText"
        @input="handleInput"
        focus
      />
    </view>
    
    <!-- 搜索建议列表 -->
    <view class="suggestion-list" v-if="showSuggestions && searchText">
      <view 
        class="suggestion-item" 
        v-for="(item, index) in suggestions" 
        :key="index"
        @click="selectSuggestion(item)"
      >
        {{ item }}
      </view>
    </view>
  </view>

    <view class="weight-chart">
      <canvas canvas-id="weightChart" class="chart"></canvas>
    </view>

    <view class="health-card">
      <text class="title">今日健康数据</text>
      <view class="data-row">
        <view class="data-item">
          <text>摄入</text>
          <text class="value">1,200</text>
          <text>kcal</text>
        </view>
        <view class="data-item">
          <text>消耗</text>
          <text class="value">850</text>
          <text>kcal</text>
        </view>
      </view>
    </view>

    <view class="action-section">
      <navigator
        url="/pages/camera/camera"
        hover-class="navigator-hover"
        class="photo-btn"
      >
        <uni-icons type="camera" size="24" color="#fff"></uni-icons>
        <text>拍照记录饮食</text> 
      </navigator>
    </view>

    <view class="quick-actions">
      <view class="action-item" @click="showWeightInput">
        <uni-icons type="scale" size="20"></uni-icons>
        <text>记录体重</text>
      </view>
      <view class="action-item" @click="goToWater">
        <uni-icons type="water" size="20"></uni-icons>
        <text>喝水打卡</text>
      </view>
    </view>

    <view class="recent-records">
      <view class="section-title">
        <text>最近记录</text>
        <text class="more">查看更多 ></text>
      </view>
      <view class="record-list">
        <view class="record-item">
          <text>早餐</text>
          <text>350 kcal</text>
        </view>
        <view class="record-item">
          <text>午餐</text>
          <text>580 kcal</text>
        </view>
      </view>
    </view>

    <!-- 嵌入的体重记录弹窗 -->
    <uni-popup ref="weightPopup" type="bottom" @change="popupChange">
      <view class="weight-popup">
        <view class="popup-header">
          <text class="title">记录体重</text>
          <uni-icons type="close" size="24" color="#999" @click="closePopup"></uni-icons>
        </view>
        
        <view class="weight-display">
          <text class="value">{{ currentValue }}</text>
          <text class="unit">kg</text>
        </view>
        
        <view class="number-pad">
          <view class="number-btn" v-for="num in [1,2,3,4,5,6,7,8,9,'.',0]" :key="num" @click="inputNum(num)">
            {{ num }}
          </view>
          <view class="number-btn delete" @click="deleteNum">
            <uni-icons type="backspace" size="24" color="#666"></uni-icons>
          </view>
        </view>
        
        <button class="confirm-btn" :class="{ disabled: !isValid }" @click="confirmWeight">
          确认记录
        </button>
      </view>
    </uni-popup>
  </view>
</template>

<script>
export default {
  data() {
    return {
      weightData: [65.2, 64.8, 64.5, 64.3, 64.1, 63.9, 63.7],
      dates: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
      currentValue: '',
      isShow: false,
      searchText: '',
      staticSuggestions: [
        "Uniapp开发教程",
        "Vue3入门",
        "微信小程序",
        "前端开发",
        "JavaScript高级编程",
        "CSS技巧",
        "移动端适配",
        "HBuilder使用指南"
      ],
      suggestions: []
    }
  },
  onReady() {
    this.drawWeightChart()
    this.currentValue = wx.getStorageSync('lastWeight') ? wx.getStorageSync('lastWeight').toString() : ''
  },
  computed: {
    isValid() {
      return /^\d+\.?\d*$/.test(this.currentValue) && 
             parseFloat(this.currentValue) > 20 && 
             parseFloat(this.currentValue) < 200
    }
  },
  methods: {
    drawWeightChart() {
      const ctx = uni.createCanvasContext('weightChart', this)
      const chartWidth = 686
      const chartHeight = 300
      const maxWeight = Math.max(...this.weightData) + 1
      const minWeight = Math.min(...this.weightData) - 1
      
      ctx.setFillStyle('#fff')
      ctx.fillRect(0, 0, chartWidth, chartHeight)
      
      ctx.setStrokeStyle('#e5e5e5')
      ctx.setLineWidth(1)
      
      for (let i = 0; i < 5; i++) {
        const y = 50 + i * 60
        ctx.moveTo(40, y)
        ctx.lineTo(chartWidth - 20, y)
        ctx.stroke()
        
        ctx.setFontSize(20)
        ctx.setFillStyle('#999')
        ctx.fillText((maxWeight - (maxWeight - minWeight) / 4 * i).toFixed(1), 10, y + 5)
      }
      
      ctx.setStrokeStyle('#4cd964')
      ctx.setLineWidth(3)
      ctx.beginPath()
      
      this.weightData.forEach((item, index) => {
        const x = 60 + index * 100
        const y = 50 + (maxWeight - item) / (maxWeight - minWeight) * 240
        
        if (index === 0) {
          ctx.moveTo(x, y)
        } else {
          ctx.lineTo(x, y)
        }
        
        ctx.setFillStyle('#4cd964')
        ctx.beginPath()
        ctx.arc(x, y, 5, 0, 2 * Math.PI)
        ctx.fill()
        
        ctx.setFontSize(22)
        ctx.setFillStyle('#666')
        ctx.fillText(this.dates[index], x - 15, chartHeight - 10)
      })
      
      ctx.stroke()
      ctx.draw()
    },
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
      this.isShow = e.show
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
      this.currentValue = this.currentValue.slice(0, -1)
    },
    confirmWeight() {
      if (!this.isValid) {
        uni.showToast({ title: '请输入有效体重', icon: 'none' })
        return
      }
      const weight = parseFloat(this.currentValue)
      this.weightData = [...this.weightData.slice(1), weight]
      wx.setStorageSync('lastWeight', weight)
      this.drawWeightChart()
      this.closePopup()
      uni.showToast({ title: `成功记录: ${weight}kg`, icon: 'success' })
    },
     // 输入处理
     handleInput() {
      if (this.searchText) {
        this.showSuggestions = true
        // 简单过滤静态数据作为建议
        this.suggestions = this.staticSuggestions.filter(item => 
          item.toLowerCase().includes(this.searchText.toLowerCase())
        )
      } else {
        this.showSuggestions = false
      }
    },
    
    // 选择建议项
    selectSuggestion(item) {
      this.searchText = item
      this.showSuggestions = false
      // 这里可以触发搜索，暂时不做具体处理
    }
  }
}
</script>

<style>
.home-container {
  padding: 20rpx;
  background-color: #f7f8fa;
}

.search-container {
  padding: 15rpx 30rpx;
  background-color: #f8f8f8;
  position: relative;
}

.search-box {
  display: flex;
  align-items: center;
  height: 70rpx;
  background-color: #fff;
  border-radius: 35rpx;
  padding: 0 20rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.search-icon {
  margin-right: 10rpx;
}

.search-input {
  flex: 1;
  height: 100%;
  font-size: 28rpx;
  color: #333;
}

.placeholder-style {
  color: #999;
  font-size: 28rpx;
}

.suggestion-list {
  position: absolute;
  left: 30rpx;
  right: 30rpx;
  top: 100rpx;
  background-color: #fff;
  border-radius: 10rpx;
  box-shadow: 0 4rpx 12rpx #b3b3b3;
  z-index: 100;
  max-height: 500rpx;
  overflow-y: auto;
}

.suggestion-item {
  padding: 20rpx 30rpx;
  font-size: 28rpx;
  color: #333;
  border-bottom: 1rpx solid #cccccc;
}

.suggestion-item:last-child {
  border-bottom: none;
}

.suggestion-item:active {
  background-color: #f5f5f5;
}


.weight-chart {
  background: white;
  border-radius: 16rpx;
  padding: 20rpx;
  margin-bottom: 24rpx;
  box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.05);
}

.chart {
  width: 100%;
  height: 350rpx;
}

.health-card {
  background: white;
  border-radius: 16rpx;
  padding: 24rpx;
  margin-bottom: 24rpx;
  box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.05);
}

.title {
  font-size: 30rpx;
  font-weight: bold;
  margin-bottom: 20rpx;
  display: block;
}

.data-row {
  display: flex;
  justify-content: space-around;
}

.data-item {
  text-align: center;
}

.value {
  font-size: 40rpx;
  font-weight: bold;
  display: block;
  color: #4cd964;
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
  box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.05);
}

.recent-records {
  background: white;
  border-radius: 16rpx;
  padding: 24rpx;
}

.section-title {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20rpx;
}

.more {
  color: #888;
}

.record-item {
  display: flex;
  justify-content: space-between;
  padding: 16rpx 0;
  border-bottom: 1rpx solid #eee;
}

/* 体重记录弹窗样式 */
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
  width: 100%;
}

.confirm-btn.disabled {
  background: #cccccc;
  opacity: 0.7;
}
</style>