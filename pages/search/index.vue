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

    <!-- 搜索历史和热门搜索 -->
    <view class="search-suggest" v-if="!searchQuery && !hasSearched">
      <!-- 搜索历史 -->
      <view class="history-section" v-if="searchHistory.length > 0">
        <view class="section-header">
          <view class="section-title">搜索历史</view>
          <view class="clear-history" @click="clearHistory">
            <uni-icons type="trash" size="16" color="#999"></uni-icons>
          </view>
        </view>
        <view class="history-tags">
          <view
            class="history-tag"
            v-for="(item, index) in searchHistory"
            :key="index"
            @click="selectHistoryItem(item)"
          >
            {{ item }}
          </view>
        </view>
      </view>

      <!-- 热门搜索 -->
      <view class="hot-section">
        <view class="section-header">
          <view class="section-title">热门搜索</view>
        </view>
        <view class="hot-tags">
          <view
            class="hot-tag"
            v-for="(item, index) in hotSearches"
            :key="index"
            @click="selectHotItem(item)"
          >
            {{ item }}
          </view>
        </view>
      </view>
    </view>

    <!-- 搜索建议 -->
    <view class="suggest-section" v-if="searchQuery && !hasSearched && suggestions.length > 0">
      <view
        class="suggest-item"
        v-for="(item, index) in suggestions"
        :key="index"
        @click="selectSuggestion(item)"
      >
        <uni-icons type="search" size="16" color="#999"></uni-icons>
        <text class="suggest-text">{{ item }}</text>
      </view>
    </view>

    <!-- 筛选栏 -->
    <view class="filter-section" v-if="hasSearched">
      <scroll-view class="filter-scroll" scroll-x show-scrollbar="false">
        <view
          class="filter-item"
          v-for="(filter, index) in filters"
          :key="index"
          :class="{ active: currentFilter === index }"
          @click="selectFilter(index)"
        >
          {{ filter.label }}
        </view>
      </scroll-view>
    </view>

    <!-- 搜索结果 -->
    <view class="results-section" v-if="hasSearched">
      <!-- 结果统计 -->
      <view class="result-stats" v-if="searchResults.length > 0">
        找到 {{ totalResults }} 个相关结果
      </view>

      <!-- 结果列表 -->
      <view v-if="searchResults.length > 0" class="results-list">
        <view
          class="result-item"
          v-for="(item, index) in searchResults"
          :key="index"
          @click="goToExhibitionDetail(item)"
        >
          <view class="result-image-container">
            <image :src="item.image" class="result-image" mode="aspectFill"></image>
            <view class="result-badge" v-if="item.status">{{ item.status }}</view>
          </view>
          <view class="result-info">
            <view class="result-name">{{ item.name }}</view>
            <view class="result-detail">
              <view class="detail-item">
                <uni-icons type="calendar" size="12" color="#666"></uni-icons>
                <text class="detail-text">{{ item.time }}</text>
              </view>
              <view class="detail-item">
                <uni-icons type="location" size="12" color="#666"></uni-icons>
                <text class="detail-text">{{ item.address }}</text>
              </view>
            </view>
            <view class="result-footer">
              <view class="price" :class="{ 'free': item.price === '免费' }">
                {{ item.price }}
              </view>
              <view class="popularity">
                <uni-icons type="fire" size="12" color="#ff6b6b"></uni-icons>
                <text class="popularity-text">{{ item.popularity }}</text>
              </view>
            </view>
          </view>
        </view>
      </view>

      <!-- 无结果状态 -->
      <view v-else class="no-results">
        <view class="no-results-icon">
          <uni-icons type="search" size="60" color="#ccc"></uni-icons>
        </view>
        <view class="no-results-text">没有找到相关结果</view>
        <view class="no-results-tip">试试其他关键词或筛选条件</view>
      </view>
    </view>

    <!-- 加载状态 -->
    <view v-if="loading" class="loading-section">
      <view class="loading-spinner"></view>
      <text class="loading-text">正在搜索...</text>
    </view>

    <!-- 加载更多 -->
    <view v-if="hasMore && searchResults.length > 0" class="load-more" @click="loadMore">
      <text class="load-more-text">加载更多</text>
    </view>
  </view>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

// 响应式数据
const searchQuery = ref('')
const searchResults = ref([])
const searchHistory = ref([])
const suggestions = ref([])
const hasSearched = ref(false)
const loading = ref(false)
const page = ref(1)
const pageSize = 10
const hasMore = ref(true)
const totalResults = ref(0)
const currentFilter = ref(0)

// 热门搜索数据
const hotSearches = ref([
  '科技展', '汽车展', '家具展', '电子展', '文化展',
  '艺术展', '建材展', '服装展', '食品展', '医疗展'
])

// 筛选选项
const filters = ref([
  { label: '综合排序', value: 'comprehensive' },
  { label: '时间最近', value: 'time' },
  { label: '人气最高', value: 'popularity' },
  { label: '价格最低', value: 'price' },
  { label: '免费优先', value: 'free' }
])

// 模拟展会数据
const mockExhibitions = Array.from({ length: 100 }, (_, i) => ({
  id: i + 1,
  image: `https://picsum.photos/300/200?random=${i + 1}`,
  name: `${['科技创新', '国际汽车', '家具家居', '电子消费', '文化艺术', '建筑建材', '服装时尚', '食品饮料', '医疗健康', '教育培训'][i % 10]}展览会 ${i + 1}`,
  time: `2024-0${(i % 12) + 1}-${(i % 28) + 1} 至 2024-0${(i % 12) + 1}-${(i % 28) + 4}`,
  address: `${['上海', '北京', '广州', '深圳', '杭州', '成都', '武汉', '西安', '重庆', '南京'][i % 10]}国际展览中心`,
  price: i % 3 === 0 ? '免费' : `¥${(i % 5 + 1) * 20}`,
  popularity: `${(i % 20 + 1) * 0.5}万人关注`,
  status: i % 4 === 0 ? '热门' : i % 4 === 1 ? '即将开始' : i % 4 === 2 ? '推荐' : ''
}))

// 事件处理函数
const goBack = () => {
  uni.navigateBack()
}

const handleSearch = () => {
  if (!searchQuery.value.trim()) return

  // 添加到搜索历史
  addToHistory(searchQuery.value.trim())

  // 执行搜索
  performSearch(true)
}

const handleInputChange = () => {
  // 实时搜索建议
  if (searchQuery.value.trim()) {
    generateSuggestions()
  } else {
    suggestions.value = []
  }
}

const clearSearch = () => {
  searchQuery.value = ''
  suggestions.value = []
  hasSearched.value = false
  searchResults.value = []
}

const clearHistory = () => {
  searchHistory.value = []
  uni.setStorageSync('searchHistory', [])
}

const selectHistoryItem = (item) => {
  searchQuery.value = item
  handleSearch()
}

const selectHotItem = (item) => {
  searchQuery.value = item
  handleSearch()
}

const selectSuggestion = (item) => {
  searchQuery.value = item
  handleSearch()
}

const selectFilter = (index) => {
  currentFilter.value = index
  performSearch(true)
}

const performSearch = (reset = false) => {
  if (loading.value || !searchQuery.value.trim()) return

  loading.value = true
  hasSearched.value = true

  if (reset) {
    page.value = 1
    searchResults.value = []
    hasMore.value = true
  }

  // 模拟API请求
  setTimeout(() => {
    const query = searchQuery.value.toLowerCase().trim()
    let results = mockExhibitions.filter(item =>
      item.name.toLowerCase().includes(query) ||
      item.address.toLowerCase().includes(query)
    )

    // 根据筛选条件排序
    const filter = filters.value[currentFilter.value]
    switch (filter.value) {
      case 'time':
        results.sort((a, b) => new Date(b.time) - new Date(a.time))
        break
      case 'popularity':
        results.sort((a, b) => parseFloat(b.popularity) - parseFloat(a.popularity))
        break
      case 'price':
        results.sort((a, b) => {
          const priceA = a.price === '免费' ? 0 : parseInt(a.price.replace('¥', ''))
          const priceB = b.price === '免费' ? 0 : parseInt(b.price.replace('¥', ''))
          return priceA - priceB
        })
        break
      case 'free':
        results.sort((a, b) => {
          if (a.price === '免费' && b.price !== '免费') return -1
          if (a.price !== '免费' && b.price === '免费') return 1
          return 0
        })
        break
    }

    totalResults.value = results.length

    // 分页处理
    const start = (page.value - 1) * pageSize
    const end = page.value * pageSize
    const pageResults = results.slice(start, end)

    if (reset) {
      searchResults.value = pageResults
    } else {
      searchResults.value = [...searchResults.value, ...pageResults]
    }

    hasMore.value = end < results.length
    if (!reset) page.value++

    loading.value = false
  }, 800)
}

const loadMore = () => {
  if (!hasMore.value || loading.value) return
  performSearch(false)
}

const generateSuggestions = () => {
  const query = searchQuery.value.toLowerCase()
  suggestions.value = mockExhibitions
    .filter(item => item.name.toLowerCase().includes(query))
    .slice(0, 5)
    .map(item => item.name)
}

const addToHistory = (query) => {
  if (!searchHistory.value.includes(query)) {
    searchHistory.value.unshift(query)
    if (searchHistory.value.length > 10) {
      searchHistory.value = searchHistory.value.slice(0, 10)
    }
    uni.setStorageSync('searchHistory', searchHistory.value)
  }
}

const goToExhibitionDetail = (item) => {
  uni.navigateTo({
    url: `/pages/exhibition/detail?id=${item.id}`
  })
}

// 页面生命周期
onMounted(() => {
  // 获取搜索历史
  const history = uni.getStorageSync('searchHistory') || []
  searchHistory.value = history

  // 获取URL参数
  const pages = getCurrentPages()
  const currentPage = pages[pages.length - 1]
  const options = currentPage.options

  if (options.query) {
    searchQuery.value = decodeURIComponent(options.query)
    handleSearch()
  }
})

// 监听搜索词变化
watch(searchQuery, (newVal) => {
  if (!newVal.trim()) {
    hasSearched.value = false
    searchResults.value = []
    suggestions.value = []
  }
})
</script>

<style scoped>
.container {
  min-height: 100vh;
  background: #f8f9fa;
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

.search-suggest {
  background: white;
  padding: 20px 16px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.section-title {
  font-size: 16px;
  font-weight: bold;
  color: #333;
}

.clear-history {
  padding: 4px;
}

.history-tags,
.hot-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.history-tag,
.hot-tag {
  padding: 6px 12px;
  background: #f0f0f0;
  border-radius: 16px;
  font-size: 14px;
  color: #666;
}

.hot-tag {
  background: #e3f2fd;
  color: #007AFF;
}

.history-section {
  margin-bottom: 24px;
}

.suggest-section {
  background: white;
  border-top: 1px solid #f0f0f0;
}

.suggest-item {
  display: flex;
  align-items: center;
  padding: 16px;
  border-bottom: 1px solid #f0f0f0;
}

.suggest-text {
  margin-left: 12px;
  font-size: 14px;
  color: #333;
}

.filter-section {
  background: white;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
}

.filter-scroll {
  padding: 0 16px;
  height: 40px; /* 固定高度 */
  overflow: hidden; /* 隐藏溢出内容 */
  white-space: nowrap; /* 防止换行 */
}

.filter-item {
  display: inline-block; /* 水平排列 */
  padding: 6px 16px;
  margin-right: 12px;
  background: #f8f9fa;
  border-radius: 20px;
  font-size: 14px;
  color: #666;
  white-space: nowrap; /* 防止文字换行 */
}

.filter-item.active {
  background: #007AFF;
  color: white;
}

.results-section {
  background: white;
  min-height: 400px;
}

.result-stats {
  padding: 16px;
  font-size: 14px;
  color: #666;
  border-bottom: 1px solid #f0f0f0;
}

.results-list {
  padding: 0 16px;
}

.result-item {
  display: flex;
  padding: 16px 0;
  border-bottom: 1px solid #f0f0f0;
}

.result-image-container {
  position: relative;
  margin-right: 12px;
}

.result-image {
  width: 80px;
  height: 60px;
  border-radius: 8px;
}

.result-badge {
  position: absolute;
  top: -4px;
  right: -4px;
  padding: 2px 6px;
  background: #007AFF;
  color: white;
  font-size: 10px;
  border-radius: 8px;
}

.result-info {
  flex: 1;
}

.result-name {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 8px;
}

.result-detail {
  display: flex;
  flex-direction: column;
  gap: 4px;
  margin-bottom: 8px;
}

.detail-item {
  display: flex;
  align-items: center;
}

.detail-text {
  font-size: 12px;
  color: #666;
  margin-left: 4px;
}

.result-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.price {
  font-size: 14px;
  font-weight: bold;
  color: #007AFF;
}

.price.free {
  color: #4CAF50;
}

.popularity {
  display: flex;
  align-items: center;
}

.popularity-text {
  font-size: 12px;
  color: #999;
  margin-left: 4px;
}

.no-results {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 60px 20px;
}

.no-results-icon {
  margin-bottom: 16px;
}

.no-results-text {
  font-size: 16px;
  color: #999;
  margin-bottom: 8px;
}

.no-results-tip {
  font-size: 14px;
  color: #ccc;
}

.loading-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 20px;
}

.loading-spinner {
  width: 32px;
  height: 32px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #007AFF;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 16px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-text {
  font-size: 14px;
  color: #666;
}

.load-more {
  padding: 16px;
  text-align: center;
  background: white;
  border-top: 1px solid #f0f0f0;
}

.load-more-text {
  font-size: 14px;
  color: #007AFF;
}
</style>
