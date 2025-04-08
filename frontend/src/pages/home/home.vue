<template>
  <view class="home-container">
    <SearchBar />
    <WeightChart />
    <HealthCard />
    <ButtonGroup />
    <RecentRecord />
  </view>
</template>

<script>
import HealthCard from './components/HealthCard.vue'
import ButtonGroup from './components/ButtonGroup.vue'
import RecentRecord from './components/RecentRecord.vue'
import SearchBar from './components/SearchBar.vue'
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
      suggestions: [],
    }
  },
  onReady() {
    this.currentValue = wx.getStorageSync('lastWeight') ? wx.getStorageSync('lastWeight').toString() : ''
  },
  computed: {
    isValid() {
      return /^\d+\.?\d*$/.test(this.currentValue) &&
        parseFloat(this.currentValue) > 20 &&
        parseFloat(this.currentValue) < 200
    },
  },
  methods: {
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
</style>