<template>
  <div class="video-container">
    <swiper
      :direction="'vertical'"
      :slides-per-view="1"
      :mousewheel="true"
      @slide-change="handleSlideChange"
      class="video-swiper"
    >
      <swiper-slide v-for="video in videos" :key="video.id">
        <VideoPlayer :video="video" :is-active="currentVideoId === video.id" />
      </swiper-slide>
    </swiper>
    <div v-if="isLoading" class="loading-indicator">加载中...</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { Swiper, SwiperSlide } from "vue-awesome-swiper";
import "swiper/css";
import VideoPlayer from "@/views/VideoList/components/VideoPlayer.vue";

// 1. 模拟的视频数据
const mockVideos = [
  {
    id: 1,
    title: "Vue 3 组合式 API 完全指南",
    videoUrl: "https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/720/Big_Buck_Bunny_720_10s_1MB.mp4",
    coverUrl: "https://picsum.photos/id/10/800/450",
    author: {
      name: "前端技术栈",
      avatar: "https://picsum.photos/id/64/200/200",
    },
    likeCount: 12345,
    commentCount: 432,
  },
  {
    id: 2,
    title: "CSS Grid 布局实战教程",
    videoUrl: "https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/720/Big_Buck_Bunny_720_10s_2MB.mp4",
    coverUrl: "https://picsum.photos/id/11/800/450",
    author: {
      name: "DesignMaster",
      avatar: "https://picsum.photos/id/65/200/200",
    },
    likeCount: 8765,
    commentCount: 211,
  },
];

// 2. 响应式数据
const videos = ref(mockVideos);
const currentVideoId = ref(videos.value[0]?.id);
const isLoading = ref(false);
let page = 1;

// 3. 模拟加载更多数据
const loadMoreVideos = () => {
  isLoading.value = true;
  
  // 模拟网络请求延迟
  setTimeout(() => {
    page++;
    const newVideos = Array.from({ length: 2 }, (_, i) => ({
      id: videos.value.length + 1 + i,
      title: `模拟视频 - 第 ${page} 页 - ${i + 1}`,
      videoUrl: `https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/720/Big_Buck_Bunny_720_10s_${videos.value.length + 1 + i}MB.mp4`,
      coverUrl: `https://picsum.photos/id/${20 + videos.value.length + i}/800/450`,
      author: {
        name: `模拟作者 ${Math.floor(Math.random() * 1000)}`,
        avatar: `https://picsum.photos/id/${70 + videos.value.length + i}/200/200`,
      },
      likeCount: Math.floor(Math.random() * 100000),
      commentCount: Math.floor(Math.random() * 1000),
    }));

    videos.value.push(...newVideos);
    isLoading.value = false;
  }, 1500); // 模拟1.5秒的加载时间
};

// 4. 处理滑动事件
const handleSlideChange = (swiper) => {
  const activeIndex = swiper.realIndex;
  currentVideoId.value = videos.value[activeIndex]?.id;

  // 当滑动到倒数第二张时，加载更多
  if (activeIndex >= videos.value.length - 1) {
    loadMoreVideos();
  }
};

// 5. 组件挂载时加载第一批额外数据
onMounted(() => {
  // 初始加载时就预加载一些数据
  if (videos.value.length < 5) {
    loadMoreVideos();
  }
});
</script>

<style scoped>
.video-container {
  height: 100vh;
  overflow: hidden;
}
.video-swiper {
  height: 100%;
}
.loading-indicator {
  position: fixed;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 0.9rem;
  z-index: 100;
}
</style>