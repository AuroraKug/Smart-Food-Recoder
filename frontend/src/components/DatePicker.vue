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
  props: {
    field: {
      type: String,
      default: ''
    },
    // 新增：最小年份
    minYear: {
      type: Number,
      default: 1900
    },
    // 新增：最大年份
    maxYear: {
      type: Number,
      default() {
        return new Date().getFullYear()
      }
    },
    // 新增：默认日期，格式：YYYY-MM-DD
    defaultDate: {
      type: String,
      default() {
        const now = new Date()
        const year = now.getFullYear()
        const month = String(now.getMonth() + 1).padStart(2, '0')
        const day = String(now.getDate()).padStart(2, '0')
        return `${year}-${month}-${day}`
      }
    }
  },
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
    this.initYears()
    this.initDefaultDate()
  },
  methods: {
    // 新增：打开日期选择器的方法
    open() {
      this.showPicker = true
      // 每次打开时重新初始化默认日期
      this.initDefaultDate()
    },
    initYears() {
      // 根据 props 初始化年份范围
      this.years = Array.from(
        { length: this.maxYear - this.minYear + 1 },
        (_, i) => this.minYear + i
      ).reverse() // 反转数组，让最近的年份在前面
    },
    initDefaultDate() {
      // 解析默认日期
      const [year, month, day] = this.defaultDate.split('-').map(Number)
      
      // 找到年份在数组中的索引（因为年份数组已经反转，所以需要重新计算索引）
      const yearIndex = this.years.findIndex(y => y === year)
      // 月份索引需要减1，因为月份数组是从1开始的
      const monthIndex = month - 1
      // 日期索引也需要减1
      const dayIndex = day - 1

      // 设置选中索引
      this.selectedIndex = [
        yearIndex > -1 ? yearIndex : 0,
        monthIndex,
        dayIndex
      ]
      
      // 更新天数并设置初始日期
      this.updateDays()
      this.date = this.defaultDate
    },
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