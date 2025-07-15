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

    <!-- 顶部导航栏 -->
    <view class="nav-section">
      <bubble-tabs
        :items="navItems"
        :is-active="true"
        v-model="currentNav"
        @tab-click="handleNavClick"
      ></bubble-tabs>
    </view>

    <!-- 轮播图区域 -->
    <view class="banner-section">
      <swiper 
        class="banner-swiper" 
        indicator-dots 
        indicator-color="rgba(255,255,255,0.6)"
        indicator-active-color="#007AFF"
        autoplay 
        interval="4000" 
        duration="500"
        circular
      >
        <swiper-item v-for="(banner, index) in banners" :key="index">
          <view class="banner-item" @click="handleBannerClick(banner)">
            <image :src="banner.image" class="banner-image" mode="aspectFill"></image>
            <view class="banner-overlay">
              <view class="banner-title">{{ banner.title }}</view>
              <view class="banner-desc">{{ banner.desc }}</view>
            </view>
          </view>
        </swiper-item>
      </swiper>
    </view>

    <!-- 快捷功能区 -->
    <view class="quick-actions">
      <view class="quick-item" v-for="(item, index) in quickActions" :key="index" @click="handleQuickAction(item)">
        <view class="quick-icon">
          <uni-icons :type="item.icon" size="24" :color="item.color"></uni-icons>
        </view>
        <text class="quick-text">{{ item.text }}</text>
      </view>
    </view>

    <!-- 展会列表区域 -->
    <view class="exhibition-section">
      <view class="section-header">
        <view class="section-title">
          <uni-icons type="calendar" size="20" color="#007AFF"></uni-icons>
          <text class="title-text">精选展会</text>
        </view>
        <picker @change="handlePickerChange" :value="exhibitionIndex" :range="exhibitionItems">
          <view class="picker-selector">
            <text class="picker-text">{{ selectedExhibitionItem }}</text>
            <uni-icons type="arrowdown" size="14" color="#666"></uni-icons>
          </view>
        </picker>
        <view class="more-btn" @click="showMore">
          <text class="more-text">更多</text>
          <uni-icons type="arrowright" size="14" color="#007AFF"></uni-icons>
        </view>
      </view>
      
      <view class="exhibition-list">
        <view
          class="exhibition-card" 
          v-for="(item, index) in exhibitions"
          :key="index"
          @click="goToExhibitionDetail(item)"
        >
          <view class="card-image-container">
            <image :src="item.image" class="card-image" mode="aspectFill"></image>
            <view class="card-badge" v-if="item.status">{{ item.status }}</view>
          </view>
          <view class="card-content">
            <view class="card-title">{{ item.name }}</view>
            <view class="card-info">
              <view class="info-item">
                <uni-icons type="calendar" size="14" color="#666"></uni-icons>
                <text class="info-text">{{ item.time }}</text>
              </view>
              <view class="info-item">
                <uni-icons type="location" size="14" color="#666"></uni-icons>
                <text class="info-text">{{ item.address }}</text>
              </view>
            </view>
            <view class="card-footer">
              <view class="price-container">
                <text class="price" :class="{ 'free': item.price === '免费' }">{{ item.price }}</text>
              </view>
              <view class="popularity-container">
                <uni-icons type="fire" size="14" color="#ff6b6b"></uni-icons>
                <text class="popularity">{{ item.popularity }}</text>
              </view>
            </view>
          </view>
        </view>
      </view>
    </view>

    <!-- 推荐活动区域 -->
    <view class="activity-section">
      <view class="section-header">
        <view class="section-title">
          <uni-icons type="star" size="20" color="#FFD700"></uni-icons>
          <text class="title-text">推荐活动</text>
        </view>
      </view>
      <scroll-view class="activity-scroll" scroll-x show-scrollbar="false">
        <view class="activity-item" v-for="(activity, index) in activities" :key="index" @click="handleActivityClick(activity)">
          <image :src="activity.image" class="activity-image" mode="aspectFill"></image>
          <view class="activity-info">
            <text class="activity-title">{{ activity.title }}</text>
            <text class="activity-desc">{{ activity.desc }}</text>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 广告区域 -->
    <view class="ad-section" v-if="adImage">
      <image :src="adImage" mode="widthFix" class="ad-image" @click="handleAdClick"></image>
    </view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import BubbleTabs from '../../components/BubbleTabs/index.vue'

// 响应式数据
const searchQuery = ref('')
const currentNav = ref(0)
const exhibitionIndex = ref(0)

// 导航数据
const navItems = ref([
  { label: '会展全览', to: '/pages/overview/index' },
  { label: '活动指南', to: '/pages/moments/index' },
  { label: '抢票大厅', to: '/pages/moments/index' },
  { label: '本地生活', to: '/pages/moments/index' }
])

// 轮播图数据
const banners = ref([
  { 
    image: 'https://picsum.photos/800/300?random=1',
    title: '2024科技创新展',
    desc: '探索未来科技趋势'
  },
  { 
    image: 'https://picsum.photos/800/300?random=2',
    title: '国际汽车展览会',
    desc: '体验最新汽车科技'
  },
  { 
    image: 'https://picsum.photos/800/300?random=3',
    title: '文化艺术博览会',
    desc: '感受艺术魅力'
  }
])

// 快捷功能
const quickActions = ref([
  { icon: 'calendar', text: '我的日程', color: '#007AFF', action: 'schedule' },
  { icon: 'star', text: '收藏夹', color: '#FFD700', action: 'favorites' },
  { icon: 'chatboxes', text: '在线客服', color: '#4CAF50', action: 'service' },
  { icon: 'location', text: '展馆地图', color: '#FF5722', action: 'map' }
])

// 展会筛选选项
const exhibitionItems = ref(['近期展会', '热门展会', '即将开展', '往期回顾'])
const selectedExhibitionItem = ref(exhibitionItems.value[0])

// 展会列表数据
const exhibitions = ref([
  {
    id: 1,
    image: 'https://picsum.photos/300/200?random=10',
    name: '2024国际消费电子展',
    time: '2024-03-15 至 2024-03-18',
    address: '上海国家会展中心',
    price: '免费',
    popularity: '8.5万人关注',
    status: '即将开始'
  },
  {
    id: 2,
    image: 'https://picsum.photos/300/200?random=11',
    name: '中国国际家具展览会',
    time: '2024-03-20 至 2024-03-23',
    address: '广州琶洲展览馆',
    price: '¥50',
    popularity: '6.2万人关注',
    status: '热门'
  },
  {
    id: 3,
    image: 'https://picsum.photos/300/200?random=12',
    name: '新能源汽车产业展',
    time: '2024-03-25 至 2024-03-28',
    address: '北京国家会议中心',
    price: '¥80',
    popularity: '12.3万人关注',
    status: '推荐'
  }
])

// 推荐活动数据
const activities = ref([
  {
    id: 1,
    image: 'https://picsum.photos/200/120?random=20',
    title: '创业沙龙',
    desc: '与行业大咖面对面'
  },
  {
    id: 2,
    image: 'https://picsum.photos/200/120?random=21',
    title: '技术分享会',
    desc: '最新技术趋势解析'
  },
  {
    id: 3,
    image: 'https://picsum.photos/200/120?random=22',
    title: '投资路演',
    desc: '优质项目展示'
  }
])

const adImage = ref('https://picsum.photos/800/120?random=100')

// 事件处理函数
const handleSearch = () => {
  if (searchQuery.value.trim()) {
    uni.navigateTo({
		url: `/pages/search/index?query=${encodeURIComponent(searchQuery.value)}`
    })
  }
}

const handleNavClick = (item) => {
  uni.navigateTo({ url: item.to })
}

const handleBannerClick = (banner) => {
  console.log('点击轮播图:', banner)
}

const handleQuickAction = (item) => {
  console.log('点击快捷功能:', item)
  // 根据不同的action执行不同的操作
  switch(item.action) {
    case 'schedule':
      uni.navigateTo({ url: '/pages/schedule/index' })
      break
    case 'favorites':
      uni.navigateTo({ url: '/pages/favorites/index' })
      break
    case 'service':
      // 打开客服
      break
    case 'map':
      uni.navigateTo({ url: '/pages/map/index' })
      break
  }
}

const handlePickerChange = (e) => {
  exhibitionIndex.value = e.detail.value
  selectedExhibitionItem.value = exhibitionItems.value[e.detail.value]
  // 根据选择的类型重新加载数据
  loadExhibitionData()
}

const showMore = () => {
  uni.navigateTo({ url: '/pages/exhibition/list' })
}

const goToExhibitionDetail = (item) => {
  uni.navigateTo({ 
    url: `/pages/exhibition/detail?id=${item.id}` 
  })
}

const handleActivityClick = (activity) => {
  uni.navigateTo({ 
    url: `/pages/activity/detail?id=${activity.id}` 
  })
}

const handleAdClick = () => {
  console.log('点击广告')
}

const loadExhibitionData = () => {
  // 模拟根据筛选条件加载不同的展会数据
  console.log('加载展会数据:', selectedExhibitionItem.value)
}

onMounted(() => {
  // 页面加载时的初始化操作
  console.log('首页加载完成')
})
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

.nav-section {
  padding: 16px;
  background: #ffffff;
  margin-bottom: 8px;
}

.banner-section {
  margin: 0 16px 20px;
}

.banner-swiper {
  height: 180px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.banner-item {
  position: relative;
  width: 100%;
  height: 100%;
}

.banner-image {
  width: 100%;
  height: 100%;
}

.banner-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.7));
  padding: 20px 16px 16px;
}

.banner-title {
  color: white;
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 4px;
}

.banner-desc {
  color: rgba(255, 255, 255, 0.8);
  font-size: 14px;
}

.quick-actions {
  display: flex;
  justify-content: space-around;
  background: white;
  margin: 0 16px 20px;
  padding: 20px 0;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.quick-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
}

.quick-icon {
  width: 48px;
  height: 48px;
  background: #f8f9fa;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
  transition: all 0.3s;
}

.quick-item:active .quick-icon {
  transform: scale(0.95);
  background: #e9ecef;
}

.quick-text {
  font-size: 13px;
  color: #666;
}

.exhibition-section {
  margin: 0 16px 20px;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.section-title {
  display: flex;
  align-items: center;
  flex: 1;
}

.title-text {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-left: 8px;
}

.picker-selector {
  display: flex;
  align-items: center;
  padding: 6px 12px;
  background: #f8f9fa;
  border-radius: 20px;
  margin-right: 12px;
}

.picker-text {
  font-size: 14px;
  color: #666;
  margin-right: 4px;
}

.more-btn {
  display: flex;
  align-items: center;
}

.more-text {
  font-size: 14px;
  color: #007AFF;
  margin-right: 4px;
}

.exhibition-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.exhibition-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  transition: all 0.3s;
}

.exhibition-card:active {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
}

.card-image-container {
  position: relative;
  height: 120px;
}

.card-image {
  width: 100%;
  height: 100%;
}

.card-badge {
  position: absolute;
  top: 8px;
  right: 8px;
  background: #007AFF;
  color: white;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
}

.card-content {
  padding: 16px;
}

.card-title {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 12px;
}

.card-info {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.info-item {
  display: flex;
  align-items: center;
}

.info-text {
  font-size: 14px;
  color: #666;
  margin-left: 6px;
}

.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.price {
  font-size: 16px;
  font-weight: bold;
  color: #007AFF;
}

.price.free {
  color: #4CAF50;
}

.popularity-container {
  display: flex;
  align-items: center;
}

.popularity {
  font-size: 12px;
  color: #999;
  margin-left: 4px;
}

.activity-section {
  margin: 0 16px 20px;
}

.activity-scroll {
  white-space: nowrap;
}

.activity-item {
  display: inline-block;
  width: 160px;
  margin-right: 12px;
  background: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  vertical-align: top;
}

.activity-image {
  width: 100%;
  height: 100px;
}

.activity-info {
  padding: 12px;
}

.activity-title {
  font-size: 14px;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 4px;
}

.activity-desc {
  font-size: 12px;
  color: #666;
  display: block;
}

.ad-section {
  margin: 0 16px 20px;
}

.ad-image {
  width: 100%;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}
</style>