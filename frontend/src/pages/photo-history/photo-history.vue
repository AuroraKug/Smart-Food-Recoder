<template>
  <view class="history-container">
    <block v-for="(group, date) in groupedHistory" :key="date">
      <!-- 日期标题 -->
      <view class="date-header">
        <text v-if="isToday(date)">今天</text>
        <text v-else-if="isYesterday(date)">昨天</text>
        <text v-else>{{ formatDate(date) }}</text>
      </view>

      <!-- 图片网格 -->
      <view class="photo-grid">
        <view 
          class="photo-item" 
          v-for="(item, index) in group" 
          :key="index"
          @tap="navigateToResult(item)"
        >
          <image class="food-image" :src="item.imageURL" mode="aspectFill" />
          <view class="probability" v-if="item.foodName !== '非菜'">{{ item.probability }}%</view>
          <view class="food-name">{{ item.foodName }}</view>
        </view>
      </view>
    </block>
  </view>
</template>

<script>

const BASE_URL = 'https://springboot-glwv-152951-5-1353388712.sh.run.tcloudbase.com'


export default {
  data() {
    return {
      // 用于显示的数据
      historyData: [],
      // 存储原始数据
      rawHistoryData: []
    }
  },

  computed: {
    groupedHistory() {
      // 按日期分组
      const grouped = {}
      this.historyData.forEach(item => {
        if (!grouped[item.date]) {
          grouped[item.date] = []
        }
        grouped[item.date].push(item)
      })
      // 按日期降序排序
      return Object.keys(grouped)
        .sort((a, b) => new Date(b) - new Date(a))
        .reduce((acc, date) => {
          acc[date] = grouped[date]
          return acc
        }, {})
    }
  },
  onLoad() {
    this.fetchSearchHistory();
  },
  methods: {
    async fetchSearchHistory() {
      try {
        const res = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/photo/searchHistory',
            method: 'GET',
            header: {
              'Authorization': 'Bearer ' + uni.getStorageSync('token'),
              'Content-Type': 'application/json'
            },
            success: resolve,
            fail: reject
          });
        });
        
        // 使用JSON方法进行深拷贝
        this.rawHistoryData = JSON.parse(JSON.stringify(res.data));
        console.log('rawHistoryData', this.rawHistoryData)
        // 处理用于显示的数据
        const processedData = res.data.map(item => {
          const foodCandidates = item.foodCandidates || {};
          let maxProb = 0;
          let foodName = '非菜';

          if (foodCandidates.result && foodCandidates.result.length > 0) {
            maxProb = (parseFloat(foodCandidates.result[0].probability) * 100).toFixed(2)
            foodName = foodCandidates.result[0].name
          }
          
          return {
            id: item.id,
            imageURL: item.imageURL || '',
            foodName: foodName,
            probability: foodName === '非菜' ? null : maxProb,  // 如果是"非菜"，概率设为null
            date: item.createdAt.substring(0, 10)  // 截取日期部分
          };
        });

        this.historyData = processedData;

      } catch (err) {
        console.error('获取识别历史失败：', err);
      }
    },

    isToday(date) {
      const today = new Date()
      return this.formatDate(today) === date
    },

    isYesterday(date) {
      const yesterday = new Date()
      yesterday.setDate(yesterday.getDate() - 1)
      return this.formatDate(yesterday) === date
    },

    formatDate(date) {
      if (typeof date === 'string') return date
      const year = date.getFullYear()
      const month = String(date.getMonth() + 1).padStart(2, '0')
      const day = String(date.getDate()).padStart(2, '0')
      return `${year}-${month}-${day}`
    },

    navigateToResult(item) {
      // 根据id找到对应的原始数据
      const rawItem = this.rawHistoryData.find(raw => raw.id === item.id);
      if (rawItem) {
        uni.navigateTo({
          url: `/pages/identified-result/identified-result?source=history&data=${encodeURIComponent(JSON.stringify(rawItem))}`
        });
      }
    }
  }
}
</script>

<style scoped>
.history-container {
  padding: 20rpx;
  background: #f5f5f5;
  min-height: 100vh;
}

.date-header {
  padding: 20rpx 0;
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.photo-grid {
  display: flex;
  flex-wrap: wrap;
  margin: 0 -10rpx;
}

.photo-item {
  width: calc(50% - 20rpx);
  margin: 10rpx;
  background: #fff;
  border-radius: 12rpx;
  overflow: hidden;
  position: relative;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
}

.food-image {
  width: 100%;
  height: 300rpx;
  display: block;
}

.probability {
  position: absolute;
  top: 16rpx;
  left: 16rpx;
  background: rgba(0, 0, 0, 0.6);
  color: #fff;
  padding: 4rpx 12rpx;
  border-radius: 20rpx;
  font-size: 24rpx;
}

.food-name {
  padding: 16rpx;
  text-align: center;
  font-size: 28rpx;
  color: #333;
}
</style>
