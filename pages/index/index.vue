<template>
  <view class="container">
    <!-- 搜索框 -->
    <view class="search-bar">
      <input
        class="search-input"
        placeholder="搜索去哪儿展会~"
        v-model="searchQuery"
        @confirm="performSearch"
      />
      <uni-icons type="search" size="20" class="search-icon"></uni-icons>
    </view>

    <!-- 顶部导航栏 -->
    <view class="header">
      <bubble-tabs
        :items="items"
        :is-active="false"
        v-model="current"
        @tab-click="goBubbleTabs"
      ></bubble-tabs>
    </view>

    <!-- 轮播图/广告区域 -->
    <swiper class="swiper" indicator-dots autoplay interval="3000" duration="1000">
      <swiper-item v-for="(banner, index) in banners" :key="index">
        <image :src="banner.image" class="swiper-image"></image>
      </swiper-item>
    </swiper>

    <!-- 近期展会列表 -->
    <view class="section">
      <view class="section-header">
        <picker @change="bindPickerChange" :value="exhibitionIndex" :range="exhibitionItems">
          <view class="uni-input">{{ selectedExhibitionItem }}</view>
          <uni-icons type="arrowright" size="16" class="picker-icon"></uni-icons>
        </picker>
        <text class="more" @click="showMore">
          <uni-icons type="arrowright" size="16"></uni-icons>
          更多
        </text>
      </view>
      <view class="exhibition-list">
        <view
          class="exhibition-item" v-for="(item, index) in exhibitions"
          :key="index"
          @click="goToExhibitionDetail(item)"
        >
          <image :src="item.image" class="exhibition-image"></image>
          <view class="exhibition-info">
            <view class="name">{{ item.name }}</view>
            <view class="time">{{ item.time }}</view>
            <view class="address">{{ item.address }}</view>
          </view>
          <view class="exhibition-other">
            <view class="price">{{ item.price }}</view>
            <view class="popularity">{{ item.popularity }}</view>
          </view>
        </view>
      </view>
    </view>

    <!-- 广告插入区域 -->
    <view class="ad-section">
      <image :src="adImage" mode="widthFix" class="ad-image"></image>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import BubbleTabs from '../../components/BubbleTabs/index.vue' // 引入自定义组件

// 数据定义
const current = ref(0)
const searchQuery = ref('')
const items = ref([
  { label: '会展全览', to: '/pages/overview/index' },
  { label: '活动指南', to: '/pages/moments/index' },
  { label: '抢票大厅', to: '/pages/moments/index' },
  { label: '本地生活', to: '/pages/moments/index' }
])
const banners = ref([
  { image: 'https://picsum.photos/800/300?random=1' },
  { image: 'https://picsum.photos/800/300?random=2' }
])
const exhibitionIndex = ref(0)
const exhibitionItems = ref(['近期展会', '热门展会', '即将开展'])
const exhibitions = ref([
  {
    image: 'https://picsum.photos/40/40?random=1',
    name: '会展名称1',
    time: '会展时间1',
    address: '会展地址1',
    price: '免费',
    popularity: '人气值1'
  },
  {
    image: 'https://picsum.photos/40/40?random=2',
    name: '会展名称2',
    time: '会展时间2',
    address: '会展地址2',
    price: '付费',
    popularity: '人气值2'
  }
])
const adImage = ref('https://picsum.photos/600/100?random=1')
// 选中的展览项
const selectedExhibitionItem = ref(exhibitionItems.value[exhibitionIndex.value])

const bindPickerChange = (e) => {
  exhibitionIndex.value = e.detail.value
  selectedExhibitionItem.value = exhibitionItems.value[e.detail.value]
}
// 展示更多展会的逻辑
const showMore = () => {
}
// 跳转到展会详情页的逻辑
const goToExhibitionDetail = (item) => {
}
// 处理顶部导航栏点击事件
const goBubbleTabs = (item) => {
  uni.navigateTo({ url: item.to })
}
const performSearch = () => {
  if (searchQuery.value.trim()) {
    uni.navigateTo({
      url: `/pages/search/index?query=${encodeURIComponent(searchQuery.value)}`
    })
    searchQuery.value = ''
  }
}
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
}

.header {
  width: 100%;
  display: flex;
  justify-content: center;
}

.swiper {
  width: 100%;
  height: 160px;
  margin-top: 12px;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.swiper-image {
  width: 100%;
  height: 100%;
  border-radius: 10px;
}

.section {
  width: 100%;
  margin-top: 16px;
}

.section-header {
  padding: 0 0 8px;
  border-bottom: 1px solid #ddd;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.uni-input {
  float: left;
}

.picker-icon {
  transform: rotate(90deg);
  display: block;
  float: left;
  margin-top: 4px;
  margin-left: 4px;
}

.more {
  display: flex;
  align-items: center;
  color: #666;
}

.exhibition-list {
  display: flex;
  flex-direction: column; /* 每个项目占一行 */
}

.exhibition-item {
  display: flex;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #ddd;
}

.exhibition-image {
  width: 60px;
  height: 60px;
  border-radius: 5px;
  margin-right: 10px;
}

.exhibition-info {
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

.exhibition-other {
  display: flex;
  justify-content: space-between;
  flex-direction: column;
  align-items: flex-end;
  align-self: flex-end;
  padding-bottom: 5px;
}

.exhibition-other view {
  margin-top: 5px;
}

.price,
.popularity {
  font-size: 12px;
  color: #999;
}

.ad-section {
  width: 100%;
  margin-top: 16px;
}

.ad-image {
  width: 100%;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.search-bar {
  width: 100%;
  display: flex;
  align-items: center;
  background-color: #f5f5f5;
  border-radius: 20px;
  padding: 8px 12px;
  margin-bottom: 12px;
  border: 1px solid #ddd;
  box-sizing: border-box;
}

.search-input {
  flex: 1;
  background: none;
  border: none;
  outline: none;
  font-size: 16px;
}

.search-icon {
  margin-left: 8px;
}
</style>
