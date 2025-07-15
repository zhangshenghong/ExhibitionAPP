<template>
  <view class="container">
    <!-- 状态栏占位 -->
    <view class="status-bar"></view>
    
    <!-- 顶部搜索栏 -->
    <view class="search-section">
      <view class="search-bar">
        <uni-icons type="search" size="18" color="#999" class="search-icon"></uni-icons>
        <input
          class="search-input"
          placeholder="搜索展会、活动..."
          v-model="searchQuery"
          confirm-type="search"
          @confirm="handleSearch"
        />
        <view class="search-btn" @click="handleSearch">搜索</view>
      </view>
    </view>

    <!-- 搜索结果 -->
    <view class="results-section">
      <view v-if="searchResults.length > 0" class="results-list">
        <view
          class="result-item"
          v-for="(item, index) in searchResults"
          :key="index"
          @click="goToExhibitionDetail(item)"
        >
          <image :src="item.image" class="result-image"></image>
          <view class="result-info">
            <view class="name">{{ item.name }}</view>
            <view class="time">{{ item.time }}</view>
            <view class="address">{{ item.address }}</view>
          </view>
        </view>
      </view>
      <view v-else class="no-results">
        没有找到与 "{{ searchQuery }}" 相关的展会
      </view>
    </view>

    <view v-if="loading" class="loading-text">正在加载中...</view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const page = ref(1) // 当前页码
const pageSize = 10 // 每页数据量
const hasMore = ref(true) // 是否还有更多数据
const loading = ref(false) // 加载状态
const searchQuery = ref('')
const searchResults = ref([])

// 模拟展会数据（实际应从API获取）
const mockExhibitions = Array.from({ length: 100 }, (_, i) => ({
  image: `https://picsum.photos/40/40?random=${i + 1}`,
  name: `会展名称${i + 1}`,
  time: `会展时间${i + 1}`,
  address: `会展地址${i + 1}`
}));


const fetchExhibitions = (reset = false) => {
  const query = searchQuery.value.toLowerCase().trim()

  if (loading.value || !hasMore.value && !reset) return

  loading.value = true

  // 模拟 API 请求延迟
  setTimeout(() => {
    let results = []

    if (reset) {
      // 重置搜索结果
      page.value = 1
      results = mockExhibitions.filter(item =>
        item.name.toLowerCase().includes(query)
      )
    } else {
      // 分页加载
      const start = (page.value - 1) * pageSize
      const end = page.value * pageSize
      results = mockExhibitions
        .filter(item => item.name.toLowerCase().includes(query))
        .slice(start, end)
    }

    if (reset) {
      searchResults.value = [...results]
    } else {
      searchResults.value = [...searchResults.value, ...results]
    }

    // 判断是否还有更多数据
    if (results.length < pageSize) {
      hasMore.value = false
    } else {
      hasMore.value = true
      page.value++
    }

    loading.value = false
  }, 500)
}


const goToExhibitionDetail = (item) => {
  // 跳转到展会详情页
  uni.navigateTo({
    url: `/pages/detail/index?id=${item.id || encodeURIComponent(item.name)}`
  })
}

// 获取初始搜索词并执行搜索
onMounted(() => {
  const pages = getCurrentPages()
  const currentPage = pages[pages.length - 1]
  const options = currentPage.options

  if (options.query) {
    searchQuery.value = decodeURIComponent(options.query)
    fetchExhibitions(true)
  }
})

const onPageScroll = () => {
  // 判断是否滚动到底部，并且还有数据可加载
  if (!loading.value && hasMore.value) {
    const query = searchQuery.value.toLowerCase().trim()
    if (query) {
      fetchExhibitions()
    }
  }
}
</script>

<style scoped>
.container {
  min-height: 100vh;
  background: linear-gradient(180deg, #f8f9fa 0%, #ffffff 100%);
}

.status-bar {
  height: var(--status-bar-height, 44px);
  background: #007AFF;
}

.search-section {
  padding: 16px;
  background: #007AFF;
}

.search-bar {
  display: flex;
  align-items: center;
  background: #ffffff;
  border-radius: 25px;
  padding: 8px 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.search-icon {
  margin-right: 8px;
}

.search-input {
  flex: 1;
  font-size: 16px;
  color: #333;
}

.search-btn {
  padding: 6px 16px;
  background: #007AFF;
  color: white;
  border-radius: 15px;
  font-size: 14px;
  margin-left: 8px;
}

.results-section {
  width: 100%;
}

.results-list {
  display: flex;
  flex-direction: column;
}

.result-item {
  display: flex;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #ddd;
}

.result-image {
  width: 60px;
  height: 60px;
  border-radius: 5px;
  margin-right: 10px;
}

.result-info {
  flex: 1;
}

.name {
  font-size: 14px;
  font-weight: bold;
  color: #333;
}

.time,
.address {
  font-size: 12px;
  color: #666;
  margin: 5px 0;
}

.no-results {
  text-align: center;
  color: #999;
  margin-top: 20px;
}

.loading-text {
  text-align: center;
  color: #999;
  margin: 10px 0;
}
</style>
