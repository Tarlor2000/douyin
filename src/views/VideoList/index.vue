<template>
  <div class="video-container">
    <swiper
      :direction="'vertical'"
      :slides-per-view="1"
      :mousewheel="true"
      @slide-change="handleSlideChange"
      @swiper="setSwiperInstance"
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
import { ref, onMounted, onUnmounted } from "vue";
import { Swiper, SwiperSlide } from "vue-awesome-swiper";
import "swiper/css";
import VideoPlayer from "@/views/VideoList/components/VideoPlayer.vue";

// 1. 模拟的视频数据
const mockVideos = [
  {
    id: 1,
    title: "Vue 3 组合式 API 完全指南",
    
    videoUrl: "https://demo.unified-streaming.com/k8s/features/stable/video/tears-of-steel/tears-of-steel.mp4/.m3u8",
    coverUrl: "https://picsum.photos/id/10/800/450",
    author: {
      name: "前端技术栈",
      avatar: "https://picsum.photos/id/64/200/200",
    },
    likeCount: 1463,
    commentCount: 2,
    collectionCount: 10,
    shareCount: 200,
    commentList: [
      {
        id: "c1",
        avatar: "https://picsum.photos/id/30/100/100",
        author: "网友小A",
        content: "这个教程太实用了！声音很清晰～",
        time: "1小时前",
        replylikeCount: 546,
        replyCount: 3,

        replies: [
          {
            avatar: "https://picsum.photos/id/33/100/100",
            author: "网友小C",
            content: "请问哪里可以获取源码？",
            time: "50分钟前",
            replylikeCount: 899,
          },
          {
            avatar: "https://picsum.photos/id/64/100/100",
            author: "前端技术栈",
            content: "源码在 GitHub 仓库，简介里有链接～",
            time: "30分钟前",
            replylikeCount: 89,
          },
          {
            avatar: "https://picsum.photos/id/33/100/100",
            author: "网友",
            content: "hi？",
            time: "23分钟前",
            replylikeCount: 68,
          },
        ],
        isShowReplies: false,
      },
      {
        id: "c2",
        avatar: "https://picsum.photos/id/31/100/100",
        author: "前端学习者",
        content: "请问这个效果是用什么插件实现的？",
        time: "2小时前",
        replylikeCount: 1,
        replyCount: 2,
        replies: [
          {
            avatar: "https://picsum.photos/id/99/100/100",
            author: "我",
            content: "用的是 vue-awesome-swiper 哦～",
            time: "1小时前",
            replylikeCount: 4,
          },
          {
            avatar: "https://picsum.photos/id/64/100/100",
            author: "前端技术栈",
            content: "源码在 GitHub 仓库，简介里有链接～",
            time: "30分钟前",
            replylikeCount: 0,
          },
        ],
        isShowReplies: false,
      },
    ],
  },
  {
    id: 2,
    title: "CSS Grid 布局实战教程",
    // 带声音的测试视频2：环境音+讲解
    videoUrl: "https://test-videos.co.uk/vids/jellyfish/mp4/h264/720/Jellyfish_720_10s_3MB.mp4",
    coverUrl: "https://picsum.photos/id/11/800/450",
    author: {
      name: "DesignMaster",
      avatar: "https://picsum.photos/id/65/200/200",
    },
    likeCount: 8765,
    commentCount: 1,
    collectionCount: 13,
    shareCount: 110,
    commentList: [
      {
        id: "c3",
        avatar: "https://picsum.photos/id/32/100/100",
        author: "UI设计师",
        content: "Grid 布局真的太灵活了！声音很清楚～",
        time: "3小时前",
        replylikeCount: 141,
        replyCount: 0,
        replies: [],
        isShowReplies: false,
      },
    ],
  },
];
// 2. 响应式数据
const videos = ref(mockVideos);
const currentVideoId = ref(videos.value[0]?.id);
const isLoading = ref(false);
const swiperInstance = ref(null);
let page = 1;

// 3. 模拟加载更多数据
const loadMoreVideos = () => {
  isLoading.value = true;
  
  setTimeout(() => {
    page++;
    const newVideos = Array.from({ length: 2 }, (_, i) => {
      // 测试视频列表
      const soundVideoUrls = [
        "https://test-videos.co.uk/vids/bee/mp4/h264/720/Bee_720_10s_1MB.mp4", // 蜜蜂嗡嗡声
        "https://test-videos.co.uk/vids/crow/mp4/h264/720/Crow_720_10s_2MB.mp4", // 乌鸦叫声
        "https://test-videos.co.uk/vids/dove/mp4/h264/720/Dove_720_10s_1MB.mp4"  // 鸽子叫声
      ];
      const randomVideoUrl = soundVideoUrls[Math.floor(Math.random() * soundVideoUrls.length)];

      // 随机生成 1-3 条主评论
      const commentCount = Math.floor(Math.random() * 3) + 1;
      const commentList = Array.from({ length: commentCount }, (_, j) => {
        const replyCount = Math.floor(Math.random() * 3);
        const replies = Array.from({ length: replyCount }, (_, r) => ({
          avatar: `https://picsum.photos/id/${40 + r}/100/100`,
          author: `网友${String.fromCharCode(70 + r)}`,
          content: `回复内容 ${r + 1}：视频声音很清晰！`,
          time: `${Math.floor(Math.random() * 5) + 1}小时前`,
        }));

        return {
          id: `c${page}${j}`,
          avatar: `https://picsum.photos/id/${30 + j}/100/100`,
          author: `网友${String.fromCharCode(65 + j)}`,
          content: `模拟评论 ${j + 1}：收获很大，视频声音也很清楚！`,
          time: `${Math.floor(Math.random() * 10) + 1}小时前`,
          replyCount: replies.length,
          replies: replies,
          isShowReplies: false,
        };
      });

      return {
        id: videos.value.length + 1 + i,
        title: `模拟视频 - 第 ${page} 页 - ${i + 1}（带声音）`,
        videoUrl: randomVideoUrl, // 赋值带声音的视频地址
        coverUrl: `https://picsum.photos/id/${20 + videos.value.length + i}/800/450`,
        author: {
          name: `模拟作者 ${Math.floor(Math.random() * 1000)}`,
          avatar: `https://picsum.photos/id/${70 + videos.value.length + i}/200/200`,
        },
        likeCount: Math.floor(Math.random() * 100000),
        commentCount: commentCount,
        commentList: commentList,
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

// 设置 Swiper 实例
const setSwiperInstance = (swiper) => {
  swiperInstance.value = swiper;
};

// 处理键盘事件
const handleKeyDown = (event) => {
  // 如果用户在输入框中，不处理键盘事件
  if (
    event.target.tagName === "INPUT" ||
    event.target.tagName === "TEXTAREA" ||
    event.target.isContentEditable
  ) {
    return;
  }

  if (!swiperInstance.value) return;

  if (event.key === "ArrowUp" || event.key === "↑") {
    event.preventDefault();
    swiperInstance.value.slidePrev();
  } else if (event.key === "ArrowDown" || event.key === "↓") {
    event.preventDefault();
    swiperInstance.value.slideNext();
  }
};

// 5. 组件挂载时加载第一批额外数据并添加键盘监听
onMounted(() => {
  // 初始加载时就预加载一些数据
  if (videos.value.length < 5) {
    loadMoreVideos();
  }
  // 添加键盘事件监听
  window.addEventListener("keydown", handleKeyDown);
});

// 组件卸载时移除键盘监听
onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyDown);
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