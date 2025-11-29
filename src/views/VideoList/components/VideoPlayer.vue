<template>
  <div class="video-player" :class="{ 'with-comments': isCommentShow, 'is-fullscreen': isFullscreen }">
    <div class="video-wrapper">
      <video
        ref="videoRef"
        :src="video.videoUrl"
        :poster="video.coverUrl"
        controlsList="nodownload"
        playsinline
        autoplay
        muted
        loop
        @click="togglePlay"
        @timeupdate="handleTimeUpdate"
        @loadedmetadata="handleLoadedMetadata"
        @play="handlePlay"
        @pause="handlePause"
      ></video>
      <div class="video-actions" v-show="!isFullscreen">
        <ul>
          <li>
            <img class="action-avatar" :src="video.author.avatar" alt="author avatar" />
          </li>
          <li><i class="iconfont icon-heart-3-fill"></i></li>
          <li @click="isCommentShow = true"><i class="iconfont icon-pinglun4"></i></li>
          <li><i class="iconfont icon-xingxingman"></i></li>
          <li><i class="iconfont icon-zhuanfa"></i></li>
          <li><i class="iconfont icon-erji"></i></li>

        </ul>
      </div>
    </div>
    <!-- 进度条 -->
    <div class="progress-wrapper" v-show="!isFullscreen">
      <div class="progress-track" @click="seekVideo">
        <div class="progress-fill" :style="{ width: progress + '%' }"></div>
      </div>
      <div class="progress-time">
        <div class="progress-icons">
          <i class="iconfont icon-bofang" v-if="isVideoPaused"></i>
          <i class="iconfont icon-zanting" v-else></i>
        </div>

        {{ formatTime(currentTime) }} / {{ formatTime(duration) }}
        <div class="danmaku-wrapper">
            <input 
              type="text" 
              class="danmaku-input" 
              placeholder="说点什么..." 
            />
            <button class="danmaku-send-btn">发送</button>
        </div>
        <div class="control-bar">
          <label class="option-item">
            <input type="checkbox" class="toggle-switch" />
            <span class="toggle-slider"></span>
            <span class="option-label">连播</span>
          </label>
          <label class="option-item">
            <input type="checkbox" class="toggle-switch" />
            <span class="toggle-slider"></span>
            <span class="option-label">清屏</span>
          </label>
          <div style = "font-size:14px">智能</div>
          <div style = "font-size:14px">倍速</div>
          <div>
              <i class="iconfont icon-shaohouzaikan"></i>
          </div>
          <div class="control-icon" @click="toggleFullscreen">
              <i class="iconfont icon-24gf-volumeMiddle"></i>
          </div>
          <div class="control-icon" @click="toggleFullscreen">
            <i class="iconfont icon-lujing-20"></i>
          </div>
          <div class="control-icon" @click="toggleFullscreen">
            <i class="iconfont icon-quanping"></i>
          </div>
        </div>
      </div>
    </div>

    <div class="video-info" v-show="!isFullscreen">
      <h3 class="video-title">@{{ video.author.name}} </h3>
      <div class="author-info">
        <span class="author-name">{{ video.title }}</span>
      </div>
    </div>
    <CommentPanel
      :is-show="isCommentShow"
      :video-id="video.id"
      :init-comments="localComments"
      @close="isCommentShow = false"
      @submit-comment="handleSubmitComment"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from "vue";
import CommentPanel from "./CommentPanel.vue";

const props = defineProps({
  video: {
    type: Object,
    required: true,
  },
  isActive: {
    type: Boolean,
    default: false,
  },
});

const videoRef = ref(null);
const isLiked = ref(false);
const isPaused = ref(true); // 当前播放状态
const currentTime = ref(0);//当前进度
const duration = ref(0);//总进度
const isCommentShow = ref(false);
const isFullscreen = ref(false);
const localComments = ref(props.video.commentList ? [...props.video.commentList] : []);
const progress = computed(() => {
  if (!duration.value) {
    return 0;
  }
  return ((currentTime.value / duration.value) * 100).toFixed(2);
});
const isVideoPaused = computed(() => isPaused.value);

// 点击视频时切换播放 / 暂停
const togglePlay = () => {
  if (!videoRef.value) return;
  if (isPaused.value) {
    videoRef.value.play();
  } else {
    videoRef.value.pause();
  }
};

// 点赞按钮状态切换
const handleLike = () => {
  isLiked.value = !isLiked.value;
};

// 视频元信息加载完成时记录总时长
const handleLoadedMetadata = () => {
  if (videoRef.value) {
    duration.value = videoRef.value.duration || 0;
  }
};

// 播放过程中实时更新当前时间
const handleTimeUpdate = () => {
  if (videoRef.value) {
    currentTime.value = videoRef.value.currentTime || 0;
  }
};

// 播放 / 暂停事件同步状态
const handlePlay = () => {
  isPaused.value = false;
};

const handlePause = () => {
  isPaused.value = true;
};

const handleSubmitComment = (newComment) => {
  localComments.value = [newComment, ...localComments.value];
};

// 键盘空格快捷播放 / 暂停
const handleKeyDown = (event) => {
  if (event.code !== "Space") return;
  if (!props.isActive) return;
  event.preventDefault();
  togglePlay();
};

onMounted(() => {
  window.addEventListener("keydown", handleKeyDown);
  // 监听全屏状态变化
  document.addEventListener("fullscreenchange", checkFullscreen);
  document.addEventListener("webkitfullscreenchange", checkFullscreen);
  document.addEventListener("mozfullscreenchange", checkFullscreen);
  document.addEventListener("MSFullscreenChange", checkFullscreen);
  checkFullscreen(); // 初始检查
});

// 点击进度条跳转到对应播放时间
const seekVideo = (event) => {
  if (!videoRef.value || !duration.value) return;
  const track = event.currentTarget.querySelector(".progress-track");
  if (!track) return;
  const rect = track.getBoundingClientRect();
  const clickX = event.clientX - rect.left;
  const percent = Math.min(Math.max(clickX / rect.width, 0), 1);
  const newTime = percent * duration.value;
  videoRef.value.currentTime = newTime;
  currentTime.value = newTime;
};

// 检测全屏状态
const checkFullscreen = () => {
  isFullscreen.value = !!(
    document.fullscreenElement ||
    document.webkitFullscreenElement ||
    document.mozFullScreenElement ||
    document.msFullscreenElement
  );
};

// 切换全屏状态
const toggleFullscreen = () => {
  const videoElement = videoRef.value;
  if (!videoElement) return;

  const target = videoElement.parentElement || videoElement;
  const requestFullscreen =
    target.requestFullscreen ||
    target.webkitRequestFullscreen ||
    target.mozRequestFullScreen ||
    target.msRequestFullscreen;
  const exitFullscreen =
    document.exitFullscreen ||
    document.webkitExitFullscreen ||
    document.mozCancelFullScreen ||
    document.msExitFullscreen;

  if (!document.fullscreenElement && requestFullscreen) {
    requestFullscreen.call(target);
  } else if (document.fullscreenElement && exitFullscreen) {
    exitFullscreen.call(document);
  }
};

// 将秒数格式化为 mm:ss 文本
const formatTime = (time) => {
  if (!time) return "00:00";
  const minutes = Math.floor(time / 60)
    .toString()
    .padStart(2, "0");
  const seconds = Math.floor(time % 60)
    .toString()
    .padStart(2, "0");
  return `${minutes}:${seconds}`;
};

// 监听 isActive 属性变化，控制播放状态
watch(
  () => props.isActive,
  (newVal) => {
    if (videoRef.value) {
      if (newVal) {
        videoRef.value.play();
      } else {
        videoRef.value.pause();
        isCommentShow.value = false;
      }
    }
  }
);

onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyDown);
  document.removeEventListener("fullscreenchange", checkFullscreen);
  document.removeEventListener("webkitfullscreenchange", checkFullscreen);
  document.removeEventListener("mozfullscreenchange", checkFullscreen);
  document.removeEventListener("MSFullscreenChange", checkFullscreen);
  if (videoRef.value) {
    videoRef.value.pause();
  }
});

// 当 videoRef 发生变化时重置进度
watch(
  () => videoRef.value,
  () => {
    currentTime.value = 0;
    duration.value = videoRef.value?.duration || 0;
  }
);

watch(
  () => props.video.commentList,
  (newVal) => {
    localComments.value = newVal ? [...newVal] : [];
  },
  { deep: true }
);
</script>

<style scoped>
.video-player {
  position: relative;
  width: 96%;
  height: 80vh;
  background-color: rgb(22,24,35);
  transition: width 0.3s ease, padding-right 0.3s ease, height 0.3s ease;
}

.video-player.is-fullscreen {
  width: 100%;
  height: 100vh;
  padding-right: 0 !important;
}

.video-player.with-comments {
  padding-right: 360px;
}

.video-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
}

video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 15px;
}

.progress-wrapper {
  position: absolute;
  bottom: -39px;
  left: 0;
  right: 0;
  display: flex;
  flex-direction: column;
  gap: 6px;
  color: #fff;
  font-size: 0.85rem;
  z-index: 15;
}

.progress-track {
  width: 100%;
  height: 2px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: rgba(255, 255, 255, 0.5);
  transition: width 0.1s linear;
}

.progress-time {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  font-variant-numeric: tabular-nums;
  font-size:16px;
  gap: 10px;
}

.control-bar {
  margin-left: auto;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 12px;
  flex-wrap: nowrap;
}

.danmaku-wrapper {
  display: flex;
  align-items: center;
  margin: 0 150px;
  background-color: rgba(255, 255, 255, 0.15);
  border-radius: 10px;
  overflow: hidden;
  transition: all 0.3s ease;
}

.danmaku-wrapper:focus-within {
  background-color: rgba(255, 255, 255, 0.25);
  border-color: rgba(255, 255, 255, 0.5);
  box-shadow: 0 0 8px rgba(255, 255, 255, 0.2);
}

.danmaku-input {
  padding: 6px 12px;
  background-color: bgc(35, 34, 33);
  border: none;
  color: #fff;
  font-size: 0.85rem;
  outline: none;
  width: 380px;
}

.danmaku-input::placeholder {
  color: rgba(255, 255, 255, 0.6);
}

.danmaku-send-btn {
  padding: 6px 16px;
  background-color: rgb(63, 63, 62);
  border-radius: 20px;
  border: none;
  color: #fff;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.3s ease;
  white-space: nowrap;
}

.danmaku-send-btn:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.danmaku-send-btn:active {
  background-color: rgba(255, 255, 255, 0.2);
}

.option-item {
  display: flex;
  align-items: center;
  gap: 4px;
  cursor: pointer;
  user-select: none;
  position: relative;
}

.toggle-switch {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 20px;
}

.toggle-slider {
  position: relative;
  display: inline-block;
  width: 36px;
  height: 20px;
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 10px;
  transition: background-color 0.3s ease;
  cursor: pointer;
}

.toggle-slider::before {
  content: "";
  position: absolute;
  width: 16px;
  height: 16px;
  left: 2px;
  top: 2px;
  background-color: #fff;
  border-radius: 50%;
  transition: transform 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.toggle-switch:checked + .toggle-slider {
  background-color: rgba(255, 255, 255, 0.8);
}

.toggle-switch:checked + .toggle-slider::before {
  transform: translateX(16px);
}

.option-label {
  color: #fff;
  font-size: 0.85rem;
  white-space: nowrap;
}

.option-item:hover .option-label {
  color: rgba(255, 255, 255, 0.9);
}

.control-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  cursor: pointer;
  transition: opacity 0.2s ease;
}

.control-icon:hover {
  opacity: 0.8;
}

.video-actions {
  position: absolute;
  top: 50%;
  right: 24px;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 101;
}

/* 当有评论区时，right 值保持不变，因为它会相对于收缩后的视频内容区域 */
.video-player.with-comments .video-actions {
  /* right 值保持不变，因为 padding-right 已经让内容区域收缩了 */
  /* 如果需要微调，可以调整这里的值 */
}

.video-actions ul {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 18px;
  align-items: center;
}
.video-actions ul li i{
  font-size:30px;
}
.video-actions li {
  font-size: 32px;
  color: #fff;
  cursor: pointer;
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.video-actions li:hover {
  transform: translateY(-3px) scale(1.05);
  opacity: 0.85;
}

.action-avatar {
  width: 35px;
  height: 35px;
  border-radius: 50%;
  border: 2px solid rgba(255, 255, 255, 0.7);
  object-fit: cover;
  display: block;
}
.video-info {
  position: absolute;
  bottom: 20px;
  left: 20px;
  color: white;
  text-shadow: 0 0 2px rgba(0, 0, 0, 0.8);
  z-index: 10;
}

.video-title {
  font-size: 1.2rem;
  font-weight: bold;
  margin-bottom: 10px;
}

.author-info {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
}

.author-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 2px solid white;
}

.interaction-buttons {
  display: flex;
  gap: 20px;
}

.interaction-buttons button {
  background: transparent;
  border: none;
  color: white;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  font-size: 0.9rem;
}


</style>