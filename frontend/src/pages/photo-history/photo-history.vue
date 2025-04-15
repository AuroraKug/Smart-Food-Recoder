<template>
  <view class="history-container">
    <block v-for="(group, date) in groupedHistory" :key="date">
      <!-- 日期标题 -->
      <view class="date-header">
        <text v-if="isToday(date)">Today</text>
        <text v-else-if="isYesterday(date)">Yesterday</text>
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
          <image class="food-image" :src="item.imageUrl" mode="aspectFill" />
          <view class="probability">{{ item.probability }}%</view>
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
      // 模拟数据
      historyData: [
        //{
        //  id: 1,
        //  imageUrl: '/static/images/food1.jpg',
        //  foodName: 'Diced Beef',
        //  probability: 95,
        //  date: '2025-04-09',
        //  searchResult: {/* 搜索结果数据 */}
        //},
        //{
        //  id: 2,
        //  imageUrl: '/static/images/food2.jpg',
        //  foodName: 'Cola',
        //  probability: 88,
        //  date: '2025-04-09',
        //  searchResult: {/* 搜索结果数据 */}
        //},
        //{
        //  id: 3,
        //  imageUrl: '/static/images/food3.jpg',
        //  foodName: 'Salad',
        //  probability: 92,
        //  date: '2025-04-08',
        //  searchResult: {/* 搜索结果数据 */}
        //},
        // 更多数据...
      ]
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
        // console.log('获取识别历史成功：', res.data);
        // 处理每条记录：选出概率最高的食物名和概率
        const processedData = res.data.map(item => {
          const foodCandidates = item.foodCandidates || {};
          let maxProb = 0;
          let foodName = '非菜';

          // // 仅保留 key 是字符串、value 是 number 类型，排除像 log_id 这种字段
          // for (const [name, prob] of Object.entries(foodCandidates)) {
          //   if (typeof prob !== 'number') continue;  // ⛔ 跳过非数字的字段

          //   if (prob > maxProb) {
          //     maxProb = prob;
          //     foodName = name;
          //   }
          // }
          if (foodCandidates.result && foodCandidates.result.length > 0) {
            // 设置主要结果
            this.mainResult = foodCandidates.result[0]
            console.log('foodCandidates.mainFesult:', this.mainResult)
            maxProb = (parseFloat(this.mainResult.probability) * 100).toFixed(2)
            foodName = this.mainResult.name
          }


          return {
            id: item.id,
            imageUrl: item.imageBase64 ? 'data:image/jpeg;base64,' + item.imageBase64 : '', // Base64转图片
            foodName: foodName,
            probability: maxProb,
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
      uni.setStorageSync('currentImage', item.imageBase64 ? 'data:image/jpeg;base64,' + item.imageBase64 : '')
      // 跳转到识别结果页面
      uni.navigateTo({
        //???
        url: `/pages/identified-result/identified-result?id=${item.id}`
      })
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
