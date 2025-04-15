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
      rawHistoryData: [],
      // 分页相关
      currentPage: 0,
      pageSize: 2,
      hasMore: true,
      isLoading: false
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
  // 添加触底加载更多
  onReachBottom() {
    if (this.hasMore && !this.isLoading) {
      this.currentPage++
      this.fetchSearchHistory()
    }
  },
  methods: {
    async fetchSearchHistory() {
      if (this.isLoading) return
      this.isLoading = true
      
      try {
        const res = await new Promise((resolve, reject) => {
          uni.request({
            url: BASE_URL + '/api/photo/searchHistory',
            method: 'GET',
            data: {
              page: this.currentPage,
              size: this.pageSize
            },
            header: {
              'Authorization': 'Bearer ' + uni.getStorageSync('token'),
              'Content-Type': 'application/json'
            },
            success: resolve,
            fail: reject
          });
        });
        
        if (res.statusCode === 200) {
          const newData = res.data
          
          // 如果是第一页，重置数据
          if (this.currentPage === 0) {
            this.rawHistoryData = JSON.parse(JSON.stringify(newData))
            this.historyData = []
          } else {
            // 追加新数据到原始数据
            this.rawHistoryData = [...this.rawHistoryData, ...JSON.parse(JSON.stringify(newData))]
          }

          // 处理用于显示的数据
          const processedData = newData.map(item => {
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
              probability: foodName === '非菜' ? null : maxProb,
              date: item.createdAt.substring(0, 10)
            };
          });

          // 追加新处理的数据
          this.historyData = [...this.historyData, ...processedData]
          
          // 判断是否还有更多数据
          this.hasMore = newData.length === this.pageSize
        }
      } catch (err) {
        console.error('获取识别历史失败：', err);
      } finally {
        this.isLoading = false
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
