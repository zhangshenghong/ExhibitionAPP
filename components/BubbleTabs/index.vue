<template>
  <view class="bubble-tabs">
    <view
      v-for="(item, index) in items"
      :key="index"
      class="bubble-tab"
      :class="{ 'active': currentIndex === index }"
      @click="handleClick(index)"
    >
      {{ item }}
    </view>
  </view>
</template>

<script setup>
import { defineEmits, defineProps, ref } from 'vue'

const props = defineProps({
  items: {
    type: Array,
    required: true
  },
  modelValue: {
    type: Number,
    default: 0
  }
})

const emit = defineEmits(['update:modelValue'])

const currentIndex = ref(props.modelValue)

const handleClick = (index) => {
  currentIndex.value = index
  emit('update:modelValue', index)
}
</script>

<style scoped>
.bubble-tabs {
  display: flex;
  justify-content: center; /* 居中对齐 */
  margin: 0 -8px;
}

.bubble-tab {
  position: relative;
  text-align: center;
  padding: 10px 15px;
  border-radius: 20px;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: all 0.3s;
  margin: 0 8px; /* 气泡项之间的间距 */
}

.bubble-tab.active {
  background-color: #007aff;
  color: #fff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.bubble-tab::before {
  content: '';
  position: absolute;
  top: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  border-bottom: 10px solid #fff;
  z-index: -1;
}

.bubble-tab.active::before {
  border-bottom-color: #007aff;
}
</style>
