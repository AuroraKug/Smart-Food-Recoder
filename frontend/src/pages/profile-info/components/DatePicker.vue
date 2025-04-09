<template>
  <view class="container">
    <!-- 遮罩层 -->
    <view v-if="showPicker" class="mask" @touchmove.prevent @click="showPicker = false"></view>

    <!-- 日期选择器 -->
    <view class="picker-container" :class="{ show: showPicker }" @click.stop>
      <view class="picker-header">
        <uni-icons type="close" size="24" color="#666" @tap="showPicker = false"></uni-icons>
      </view>

      <picker-view class="picker-view" :value="selectedIndex" @change="bindChange" indicator-class="indicator">
        <picker-view-column>
          <view class="picker-item" v-for="(year, index) in years" :key="index">{{ year }}年</view>
        </picker-view-column>
        <picker-view-column>
          <view class="picker-item" v-for="(month, index) in months" :key="index">{{ month }}月</view>
        </picker-view-column>
        <picker-view-column>
          <view class="picker-item" v-for="(day, index) in days" :key="index">{{ day }}日</view>
        </picker-view-column>
      </picker-view>

      <button class="save-btn" @click="handleSave">保存</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      showPicker: false,
      date: '',
      years: [],
      months: Array.from({ length: 12 }, (_, i) => i + 1),
      days: [],
      selectedIndex: [0, 0, 0]
    }
  },
  created() {
    // 初始化年份（1900-当前年份）
    const currentYear = new Date().getFullYear()
    this.years = Array.from({ length: currentYear - 1899 }, (_, i) => 1900 + i)

    // 初始化天数
    this.updateDays()
  },
  props: {
    field: {
      type: String,
      default: ''
    }
  },
  methods: {
    bindChange(e) {
      const [yearIndex, monthIndex, dayIndex] = e.detail.value
      this.selectedIndex = [yearIndex, monthIndex, dayIndex]
      this.updateDays()
    },
    updateDays() {
      const selectedYear = this.years[this.selectedIndex[0]]
      const selectedMonth = this.months[this.selectedIndex[1]]

      // 计算当月天数
      const daysInMonth = new Date(selectedYear, selectedMonth, 0).getDate()
      this.days = Array.from({ length: daysInMonth }, (_, i) => i + 1)

      // 调整日期索引防止越界
      if (this.selectedIndex[2] >= daysInMonth) {
        this.selectedIndex[2] = daysInMonth - 1
      }
    },
    saveDate() {
      const year = this.years[this.selectedIndex[0]]
      const month = this.months[this.selectedIndex[1]].toString().padStart(2, '0')
      const day = this.days[this.selectedIndex[2]].toString().padStart(2, '0')
      this.date = `${year}-${month}-${day}`
      this.showPicker = false
    },
    handleSave() {
      this.saveDate()
      this.$emit('save', {
        field: this.field,
        date: this.date
      })
      this.showPicker = false
      console.log(this.date);

    }
  }
}
</script>

<style scoped>
.container {
  padding: 20px;
}

.input-item {
  padding: 15px;
  border-bottom: 1px solid #eee;
}

.mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 999;
}

.picker-container {
  position: fixed;
  left: 0;
  right: 0;
  bottom: -100%;
  background: #fff;
  z-index: 1000;
  transition: all 0.3s ease;
  border-radius: 10px 10px 0 0;
  padding: 15px;
}

.picker-container.show {
  bottom: 0;
}

.picker-header {
  text-align: right;
  padding: 10px;
}

.close {
  font-size: 28px;
  padding: 0 15px;
}

.picker-view {
  height: 200px;
  margin: 20px 0;
}

.indicator {
  height: 40px;
  line-height: 40px;
  background: #f5f5f5;
}

.picker-item {
  line-height: 40px;
  text-align: center;
}

.save-btn {
  margin: 15px 0;
  background: #4cd964;
  color: white;
}
</style>