<template>
  <view class="home-container">
    <SearchBar />
    <WeightChart :isCanvasVisible="isCanvasVisible" />
    <HealthCard />
    <ButtonGroup @toggle-canvas="toggleCanvas" />
    <RecentRecord />
  </view>
</template>

<script>
import SearchBar from '@/components/SearchBar.vue'
import HealthCard from './components/HealthCard.vue'
import ButtonGroup from './components/ButtonGroup.vue'
import RecentRecord from './components/RecentRecord.vue'
import WeightChart from './components/WeightChart.vue'

export default {
  components: {
    HealthCard,
    ButtonGroup,
    RecentRecord,
    SearchBar,
    WeightChart
  },
  data() {
    return {
      isCanvasVisible: true,
      isRefreshing: false
    }
  },
  methods: {
    toggleCanvas(visible) {
      this.isCanvasVisible = visible
    },
    onPullDownRefresh() {
      this.isRefreshing = true
      // 发送各个组件的刷新事件
      uni.$emit('refresh-weight-data') // 体重数据刷新
      uni.$emit('refresh-health-data') // 健康数据刷新
      uni.$emit('refresh-recent-records') // 最近记录刷新
      setTimeout(() => {
        this.isRefreshing = false
        uni.stopPullDownRefresh()
      }, 3000)
    }
  }
}
</script>

<style>
.home-container {
  padding: 20rpx;
  background-color: #f7f8fa;
}
</style>