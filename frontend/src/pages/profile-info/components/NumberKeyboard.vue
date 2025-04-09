<template>
  <view>
    <!-- 遮罩层 -->
    <view v-if="showKeyboard" class="mask" @touchmove.prevent @click="handleClose"></view>

    <!-- 键盘容器 -->
    <view class="number-keyboard-container" :class="{ show: showKeyboard }" @click.stop>
      <view class="keyboard-header">
        <text class="title">{{ title }}</text>
        <uni-icons type="close" size="24" color="#666" @click="handleClose"></uni-icons>
      </view>

      <!-- 数字显示区 -->
      <view class="number-display">
        {{ currentValue || '请输入' }}
        <text v-if="currentValue" class="unit">{{ unit }}</text>
      </view>

      <!-- 键盘主体 -->
      <view class="keyboard-body">
        <view class="number-grid">
          <view v-for="num in 9" :key="num" class="number-btn" @click="handleInput(num)">
            {{ num }}
          </view>
          <view class="number-btn" :class="{ disabled: hasDecimal }" @click="handleDecimal">
            .
          </view>
          <view class="number-btn" @click="handleInput(0)">
            0
          </view>
          <view class="number-btn delete-btn" @click="handleDelete">
            ⌫
          </view>
        </view>
      </view>

      <button class="confirm-btn" :disabled="!currentValue" @click="handleConfirm">
        确定
      </button>
    </view>
  </view>
</template>

<script>
export default {
  props: {
    field: { // 用于区分不同输入项
      type: String,
      required: true
    },
    title: { // 键盘标题
      type: String,
      default: '请输入数字'
    },
    unit: { // 单位
      type: String,
      default: ''
    },
    maxLength: { // 最大输入位数
      type: Number,
      default: 6
    }
  },
  data() {
    return {
      showKeyboard: false,
      currentValue: ''
    }
  },
  methods: {
    // 打开键盘（供父组件调用）
    open(value = '') {
      this.currentValue = value.toString()
      this.showKeyboard = true
    },

    handleInput(num) {
      if (this.currentValue.length >= this.maxLength) return
      this.currentValue += num.toString()
    },

    handleDelete() {
      this.currentValue = this.currentValue.slice(0, -1)
    },

    handleConfirm() {
      this.$emit('confirm', {
        field: this.field,
        value: this.currentValue
      })
      this.showKeyboard = false
    },

    handleClose() {
      this.showKeyboard = false
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
/* 复用日期选择器的遮罩样式 */
.mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 999;
}

.number-keyboard-container {
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

.number-keyboard-container.show {
  bottom: 0;
}

.keyboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 15px;
  border-bottom: 1px solid #eee;
}

.title {
  font-size: 16px;
  color: #333;
}

.number-display {
  height: 60px;
  line-height: 60px;
  text-align: center;
  font-size: 28px;
  color: #333;
  padding: 10px 0;
}

.unit {
  font-size: 14px;
  color: #999;
  margin-left: 5px;
}

.keyboard-body {
  padding: 10px 0;
}

.number-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.number-btn {
  height: 50px;
  line-height: 50px;
  text-align: center;
  background: #f5f5f5;
  border-radius: 8px;
  font-size: 24px;
  transition: all 0.1s;
}

.number-btn:active {
  background: #e0e0e0;
}

.delete-btn {
  font-size: 28px;
}

.placeholder {
  visibility: hidden;
}

.confirm-btn {
  margin-top: 15px;
  background: #4cd964;
  color: white;
}

.confirm-btn:disabled {
  background: #ddd;
}
</style>