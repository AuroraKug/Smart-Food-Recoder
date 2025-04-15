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
      suggestions: [],
      isRefreshing: false
    }
  },
  methods: {
    toggleCanvas(visible) {
      this.isCanvasVisible = visible // 切换 canvas 的显示状态
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