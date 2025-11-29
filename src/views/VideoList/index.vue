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
    commentCount: 2, // 评论数（与 commentList 长度一致）
    commentList: [ // 该视频的初始评论列表
      {
        avatar: "https://picsum.photos/id/30/100/100",
        author: "网友小A",
        content: "这个教程太实用了！",
        time: "1小时前",
      },
      {
        avatar: "https://picsum.photos/id/31/100/100",
        author: "前端学习者",
        content: "请问这个效果是用什么插件实现的？",
        time: "2小时前",
      },
    ],
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
    commentCount: 1,
    commentList: [
      {
        avatar: "https://picsum.photos/id/32/100/100",
        author: "UI设计师",
        content: "Grid 布局真的太灵活了！",
        time: "3小时前",
      },
    ],
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
  
  setTimeout(() => {
    page++;
    const newVideos = Array.from({ length: 2 }, (_, i) => {
      // 随机生成 1-3 条模拟评论
      const commentCount = Math.floor(Math.random() * 3) + 1;
      const commentList = Array.from({ length: commentCount }, (_, j) => ({
        avatar: `https://picsum.photos/id/${30 + j}/100/100`,
        author: `网友${String.fromCharCode(65 + j)}`,
        content: `模拟评论 ${j + 1}：这个视频不错！`,
        time: `${Math.floor(Math.random() * 10) + 1}小时前`,
      }));

      return {
        id: videos.value.length + 1 + i,
        title: `模拟视频 - 第 ${page} 页 - ${i + 1}`,
        videoUrl: `https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/720/Big_Buck_Bunny_720_10s_${videos.value.length + 1 + i}MB.mp4`,
        coverUrl: `https://picsum.photos/id/${20 + videos.value.length + i}/800/450`,
        author: {
          name: `模拟作者 ${Math.floor(Math.random() * 1000)}`,
          avatar: `https://picsum.photos/id/${70 + videos.value.length + i}/200/200`,
        },
        likeCount: Math.floor(Math.random() * 100000),
        commentCount: commentCount,
        commentList: commentList, // 绑定评论列表
      };
    });

    videos.value.push(...newVideos);
    isLoading.value = false;
  }, 1500);
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