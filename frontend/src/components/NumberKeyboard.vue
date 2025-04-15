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
        {{ displayValue }}
        <text v-if="currentValue" class="unit">{{ unit }}</text>
        <text v-if="isOutOfRange" class="error-tip">超出有效范围</text>
      </view>

      <!-- 键盘主体 -->
      <view class="keyboard-body">
        <view class="number-grid">
          <view v-for="num in 9" :key="num" class="number-btn" @click="handleInput(num.toString())">
            {{ num }}
          </view>
          <view class="number-btn" :class="{ disabled: hasDecimal }" @click="handleDecimal">
            .
          </view>
          <view class="number-btn" @click="handleInput('0')">
            0
          </view>
          <view class="number-btn delete-btn" @click="handleDelete">
            ⌫
          </view>
        </view>
      </view>

      <button class="confirm-btn" :disabled="!isValidValue || isOutOfRange" @click="handleConfirm">
        确定
      </button>
    </view>
  </view>
</template>

<script>
export default {
  props: {
    field: {
      type: String,
      required: true
    },
    title: {
      type: String,
      default: '请输入数字'
    },
    unit: {
      type: String,
      default: ''
    },
    maxLength: {
      type: Number,
      default: 6
    },
    // 新增范围限制
    range: {
      type: Array,
      default: () => [0, Infinity]
    },
    // 小数位数限制
    decimalDigits: {
      type: Number,
      default: 2
    }
  },
  data() {
    return {
      showKeyboard: false,
      currentValue: '',
      originalValue: '',
      isInputting: false
    }
  },
  computed: {
    displayValue() {
      return this.currentValue === '' ? '' : this.currentValue
    },
    hasDecimal() {
      return this.currentValue.includes('.')
    },
    isValidValue() {
      // 验证：不能为空、不能以小数点开头、不能有多个小数点
      return this.currentValue !== '' &&
        !this.currentValue.startsWith('.') &&
        (this.currentValue.match(/\./g) || []).length <= 1
    },
    isOutOfRange() {
      if (!this.currentValue || !this.isValidValue) return false
      const numValue = parseFloat(this.currentValue)
      return numValue < this.range[0] || numValue > this.range[1]
    }
  },
  methods: {
    open(value = '') {
      this.originalValue = value.toString()
      this.currentValue = value.toString()
      this.isInputting = false
      this.showKeyboard = true
    },

    handleInput(char) {
      this.isInputting = true
      // 处理数字输入
      const parts = this.currentValue.split('.')

      // 整数部分限制
      if (!this.hasDecimal) {
        if (parts[0].length >= this.maxLength) return
        // 防止前导零（0后面只能跟小数点）
        if (parts[0] === '0' && char !== '.') return
      }
      // 小数部分限制
      else {
        if (parts[1].length >= this.decimalDigits) return
      }

      this.currentValue += char
      this.$emit('input', {
        field: this.field,
        value: this.currentValue
      })
    },

    // 新增小数点处理方法
    handleDecimal() {
      if (this.currentValue === '') {
        this.currentValue = '0.'
      } else if (!this.hasDecimal) {
        this.currentValue += '.'
      }
    },

    handleDelete() {
      this.isInputting = true
      if (this.currentValue.length > 0) {
        this.currentValue = this.currentValue.slice(0, -1)
        this.$emit('input', {
          field: this.field,
          value: this.currentValue
        })
      }
    },

    handleConfirm() {
      // 格式化最终值（如 ".5" → "0.5"）
      let finalValue = this.currentValue
      if (finalValue.startsWith('.')) {
        finalValue = '0' + finalValue
      } else if (finalValue.endsWith('.')) {
        finalValue = finalValue.slice(0, -1)
      }

      this.$emit('confirm', {
        field: this.field,
        value: finalValue,
        numericValue: parseFloat(finalValue)
      })
      this.isInputting = false
      this.showKeyboard = false
    },

    handleClose() {
      if (this.isInputting) {
        this.currentValue = this.originalValue
        this.$emit('input', {
          field: this.field,
          value: this.originalValue
        })
      }
      this.isInputting = false
      this.showKeyboard = false
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
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
  position: relative;
}

.unit {
  font-size: 14px;
  color: #999;
  margin-left: 5px;
}

.error-tip {
  position: absolute;
  right: 20px;
  bottom: 5px;
  font-size: 12px;
  color: #ff0000;
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

.number-btn.disabled {
  opacity: 0.5;
  pointer-events: none;
}

.delete-btn {
  font-size: 28px;
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