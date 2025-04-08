<template>
  <view class="weight-progress-section">
    <view class="progress-wrapper">
      <view class="progress-container">
        <!-- <canvas canvas-id="progressCanvas" class="progress-canvas"></canvas> -->
        <view class="progress-info">
          <view class="progress-value">{{ weightLost }}<text class="unit">kg</text></view>
          <text class="progress-label">已减体重</text>
        </view>
      </view>
    </view>

    <view class="weight-data">
      <view class="data-item">
        <text class="data-label">初始体重</text>
        <text class="data-value">{{ initialWeight }}<text class="unit">kg</text></text>
      </view>
      <view class="data-item">
        <text class="data-label">当前体重</text>
        <text class="data-value">{{ currentWeight }}<text class="unit">kg</text></text>
      </view>
      <view class="data-item">
        <text class="data-label">目标体重</text>
        <text class="data-value">{{ targetWeight }}<text class="unit">kg</text></text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      initialWeight: 68.5,
      currentWeight: 64.2,
      targetWeight: 60.0,
    }
  },
  computed: {
    weightLost() {
      return (this.initialWeight - this.currentWeight).toFixed(1)
    },
    progressPercentage() {
      const total = this.initialWeight - this.targetWeight
      const lost = this.initialWeight - this.currentWeight
      return Math.min(100, Math.max(0, (lost / total) * 100))
    }
  },
  methods: {
    drawProgressCircle() {
      const ctx = uni.createCanvasContext('progressCanvas', this)
      const width = 350
      const height = 150
      const centerX = width / 2
      const centerY = height
      const radius = 120
      const lineWidth = 12

      ctx.clearRect(0, 0, width, height)

      ctx.beginPath()
      ctx.arc(centerX, centerY, radius, Math.PI, 0, false)
      ctx.setStrokeStyle('#f0f0f0')
      ctx.setLineWidth(lineWidth)
      ctx.setLineCap('round')
      ctx.stroke()

      const endAngle = Math.PI + (Math.PI * this.progressPercentage / 100)
      ctx.beginPath()
      ctx.arc(centerX, centerY, radius, Math.PI, endAngle, false)
      ctx.setStrokeStyle('#4cd964')
      ctx.setLineWidth(lineWidth)
      ctx.setLineCap('round')
      ctx.stroke()

      const dotX = centerX + radius * Math.cos(endAngle)
      const dotY = centerY + radius * Math.sin(endAngle)
      ctx.beginPath()
      ctx.arc(dotX, dotY, lineWidth / 2, 0, Math.PI * 2)
      ctx.setFillStyle('#4cd964')
      ctx.fill()

      ctx.draw()
    }
  },
  mounted() {
    this.drawProgressCircle();
  },
  watch: {
    currentWeight() {
      this.drawProgressCircle();
    }
  }
}

</script>

<style scoped>
.weight-progress-section {
  background: white;
  border-radius: 16rpx;
  padding: 30rpx;
  margin: 12rpx auto 24rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
  width: 90%;
}

.progress-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 20rpx;
}

.progress-container {
  position: relative;
  height: 260rpx;
  width: 600rpx;
}

.progress-canvas {
  width: 600rpx !important;
  height: 260rpx !important;
  display: block;
  margin: 0 auto;
}

.progress-info {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 60rpx;
  text-align: center;
}

.progress-value {
  font-size: 56rpx;
  font-weight: bold;
  color: #4cd964;
  line-height: 1.2;
}

.progress-label {
  font-size: 24rpx;
  color: #999;
  display: block;
}

.weight-data {
  display: flex;
  justify-content: space-around;
  margin-top: 20rpx;
}

.data-item {
  text-align: center;
  flex: 1;
}

.data-label {
  font-size: 24rpx;
  color: #999;
  display: block;
  margin-bottom: 8rpx;
}

.data-value {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.unit {
  font-size: 24rpx;
  color: #999;
  margin-left: 4rpx;
}
</style>