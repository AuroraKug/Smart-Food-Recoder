<template>
  <view class="home-container">
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
      <button class="photo-btn" @click="goToCamera">
        <uni-icons type="camera" size="24" color="#fff"></uni-icons>
        <text>拍照记录饮食</text>
      </button>
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

    <WeighInputPopup ref="weightPopup" @confirm="handleWeightConfirm" />
  </view>
</template>

<script>
import WeighInputPopup from './component/WeighInputPopup.vue';

export default {
  data() {
    return {
      weightData: [65.2, 64.8, 64.5, 64.3, 64.1, 63.9, 63.7],
      dates: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
    }
  },
  onReady() {
    this.drawWeightChart();
  },
  methods: {
    drawWeightChart() {
      const ctx = uni.createCanvasContext('weightChart', this);
      const chartWidth = 686;
      const chartHeight = 300;
      const maxWeight = Math.max(...this.weightData) + 1;
      const minWeight = Math.min(...this.weightData) - 1;
      
      // 绘制背景
      ctx.setFillStyle('#fff');
      ctx.fillRect(0, 0, chartWidth, chartHeight);
      
      // 绘制坐标轴
      ctx.setStrokeStyle('#e5e5e5');
      ctx.setLineWidth(1);
      
      // 绘制横线
      for (let i = 0; i < 5; i++) {
        const y = 50 + i * 60;
        ctx.moveTo(40, y);
        ctx.lineTo(chartWidth - 20, y);
        ctx.stroke();
        
        // 刻度值
        ctx.setFontSize(20);
        ctx.setFillStyle('#999');
        ctx.fillText((maxWeight - (maxWeight - minWeight) / 4 * i).toFixed(1), 10, y + 5);
      }
      
      // 绘制数据点
      ctx.setStrokeStyle('#4cd964');
      ctx.setLineWidth(3);
      ctx.beginPath();
      
      this.weightData.forEach((item, index) => {
        const x = 60 + index * 100;
        const y = 50 + (maxWeight - item) / (maxWeight - minWeight) * 240;
        
        if (index === 0) {
          ctx.moveTo(x, y);
        } else {
          ctx.lineTo(x, y);
        }
        
        // 绘制数据点
        ctx.setFillStyle('#4cd964');
        ctx.beginPath();
        ctx.arc(x, y, 5, 0, 2 * Math.PI);
        ctx.fill();
        
        // 日期标签
        ctx.setFontSize(22);
        ctx.setFillStyle('#666');
        ctx.fillText(this.dates[index], x - 15, chartHeight - 10);
      });
      
      ctx.stroke();
      ctx.draw();
    },
    goToCamera() {
      uni.navigateTo({
        url: '/pages/camera/camera'
      });
    },
    goToWater() {
      uni.navigateTo({
        url: '/pages/water/water'
      });
    },
    showWeightInput() {
      // 添加安全判断
      this.$nextTick(() => {
        if (this.$refs.weightPopup && this.$refs.weightPopup.open) {
          this.$refs.weightPopup.open()
        } else {
          console.error('体重弹窗组件未正确初始化')
          uni.showToast({
            title: '功能暂不可用',
            icon: 'none'
          })
        }
      })
    },
    handleWeightConfirm(weight) {
      console.log('记录的体重:', weight)
      // 这里可以添加提交逻辑
      uni.showToast({
        title: `成功记录: ${weight}kg`,
        icon: 'success'
      })
      
      // 更新图表数据示例
      this.weightData = [...this.weightData.slice(1), weight]
      this.drawWeightChart()
    }
  }
}
</script>

<style>
.home-container {
  padding: 20rpx;
  background-color: #f7f8fa;
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
</style>