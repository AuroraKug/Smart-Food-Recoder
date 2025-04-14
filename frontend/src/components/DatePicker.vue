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
    minYear: {
      type: Number,
      default: 1900
    },
    maxYear: {
      type: Number,
      default() {
        return new Date().getFullYear()
      }
    }
  },
  data() {
    return {
      showPicker: false,
      years: [],
      months: [],
      days: [],
      selectedIndex: [0, 0, 0],
      today: this.getToday()
    }
  },
  created() {
    this.initYears()
    this.initMonths()
    this.initDays()
  },
  methods: {
    // 获取东八区的今天日期
    getToday() {
      const now = new Date()
      const utc8Offset = 8 * 60 // 东八区偏移分钟数
      const localOffset = now.getTimezoneOffset() // 本地时区偏移分钟数
      const utc8Date = new Date(now.getTime() + (utc8Offset + localOffset) * 60000)
      
      return {
        year: utc8Date.getFullYear(),
        month: utc8Date.getMonth() + 1,
        day: utc8Date.getDate()
      }
    },
    
    open() {
      this.showPicker = true
    },

    initYears() {
      this.years = Array.from(
        { length: this.maxYear - this.minYear + 1 },
        (_, i) => this.minYear + i
      ).reverse()
    },

    initMonths() {
      const selectedYear = this.years[this.selectedIndex[0]]
      if (selectedYear === this.today.year) {
        // 如果是今年，只显示到当前月份
        this.months = Array.from(
          { length: this.today.month },
          (_, i) => String(i + 1).padStart(2, '0')
        )
      } else {
        // 其他年份显示所有月份
        this.months = Array.from(
          { length: 12 },
          (_, i) => String(i + 1).padStart(2, '0')
        )
      }
    },

    initDays() {
      const selectedYear = this.years[this.selectedIndex[0]]
      const selectedMonth = parseInt(this.months[this.selectedIndex[1]])
      
      // 计算当月天数
      const daysInMonth = new Date(selectedYear, selectedMonth, 0).getDate()
      
      // 如果是当年当月，只显示到今天
      if (selectedYear === this.today.year && selectedMonth === this.today.month) {
        this.days = Array.from(
          { length: this.today.day },
          (_, i) => String(i + 1).padStart(2, '0')
        )
      } else {
        this.days = Array.from(
          { length: daysInMonth },
          (_, i) => String(i + 1).padStart(2, '0')
        )
      }

      // 调整日期索引防止越界
      if (this.selectedIndex[2] >= this.days.length) {
        this.selectedIndex[2] = this.days.length - 1
      }
    },

    bindChange(e) {
      const val = e.detail.value
      
      // 更新选中索引
      this.selectedIndex = val
      
      // 重新初始化月份和日期
      this.initMonths()
      this.initDays()
      
      // 确保选中的月份和日期有效
      if (this.selectedIndex[1] >= this.months.length) {
        this.selectedIndex[1] = this.months.length - 1
      }
      if (this.selectedIndex[2] >= this.days.length) {
        this.selectedIndex[2] = this.days.length - 1
      }
    },

    handleSave() {
      const year = this.years[this.selectedIndex[0]]
      const month = this.months[this.selectedIndex[1]]
      const day = this.days[this.selectedIndex[2]]
      
      const date = `${year}-${month}-${day}`
      
      this.$emit('save', {
        field: this.field,
        date: date
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