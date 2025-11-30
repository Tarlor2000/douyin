<template>
  <div class="video-player" :class="{ 'with-comments': isCommentShow, 'is-fullscreen': isFullscreen }">
    <div class="video-wrapper">
    <video
      ref="videoRef"
      :poster="video.coverUrl"
      controlsList="nodownload"
      playsinline
      loop
      @click="togglePlay"
      @timeupdate="handleTimeUpdate"
      @loadedmetadata="handleLoadedMetadata"
      @play="handlePlay"
      @pause="handlePause"
        @error="handleVideoError"
    ></video>
      <div class="video-actions" v-show="!isFullscreen">
        <ul>
          <li>
            <img class="action-avatar" :src="video.author.avatar" alt="author avatar" />
          </li>
          <li>
            <i class="iconfont icon-heart-3-fill"></i>
            <div class="action-count">{{ video.likeCount }}</div>
          </li>
          <li @click="isCommentShow = true">
            <i class="iconfont icon-pinglun4"></i>
            <div class="action-count">{{ video.commentCount }}</div>
          </li>
          <li>
            <i class="iconfont icon-xingxingman"></i>
            <div class="action-count">{{ video.collectionCount }}</div>
          </li>
          <li>
            <i class="iconfont icon-zhuanfa"></i>
            <div class="action-count">{{ video.shareCount }}</div>
          </li>
          <li>
            <i class="iconfont icon-erji"></i>
            <div class="action-count">听抖音</div>
          </li>

        </ul>
      </div>
    <!-- 进度条 -->
      <div class="progress-wrapper" v-show="!isFullscreen">
        <div class="progress-track" @click="seekVideo">
        <div class="progress-fill" :style="{ width: progress + '%' }"></div>
      </div>
      <div class="progress-time">
        <div class="progress-icons">
          <i class="iconfont icon-bofang" v-if="isVideoPaused"  @click="togglePlay"></i>
          <i class="iconfont icon-zanting" v-else @click="togglePlay"></i>
        </div>

        <span class="time-display">{{ formatTime(currentTime) }} / {{ formatTime(duration) }}</span>
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
          <div 
            class="speed-control" 
            style = "font-size:14px"
            @mouseenter="showSpeedMenu = true"
            @mouseleave="showSpeedMenu = false"
          >
            倍速
            <div class="speed-menu" v-show="showSpeedMenu" @mouseenter="showSpeedMenu = true">
              <div 
                class="speed-option" 
                v-for="speed in speedOptions" 
                :key="speed"
                :class="{ active: currentSpeed === speed }"
                @click="handleSpeedChange(speed)"
              >
                {{ speed }}x
              </div>
            </div>
          </div>
          <div>
              <i class="iconfont icon-shaohouzaikan"></i>
        </div>

          <!-- 音量控制 -->
          <div 
            class="volume-control control-icon"
            @mouseenter="showVolumeMenu = true"
            @mouseleave="showVolumeMenu = false"
          >
            <i 
              class="iconfont" 
              :class="isMuted || volume === 0 ? 'icon-24gf-volumeCross' : 'icon-24gf-volumeMiddle'"
              @click="toggleMute"
            ></i>
            <div class="volume-menu" v-show="showVolumeMenu" @mouseenter="showVolumeMenu = true">
              <div class="volume-slider-wrapper">
                <div class="volume-value">{{ Math.round(volume * 100) }}</div>
                <input
                  type="range"
                  min="0"
                  max="1"
                  step="0.01"
                  :value="volume"
                  @input="handleVolumeInput"
                  class="volume-slider"
                />
              </div>
            </div>
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
import Hls from "hls.js";

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
const hls = ref(null);
const isLiked = ref(false);
const isPaused = ref(true); // 当前播放状态
const currentTime = ref(0);//当前进度
const duration = ref(0);//总进度
const isCommentShow = ref(false);
const isFullscreen = ref(false);
const localComments = ref(props.video.commentList ? [...props.video.commentList] : []);
const showSpeedMenu = ref(false);
const currentSpeed = ref(1.0);
const speedOptions = [0.5, 0.75, 1.0, 1.25, 1.5, 2.0];
const showVolumeMenu = ref(false);
const volume = ref(0.5);
const isMuted = ref(false);
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

// 初始化视频源
const initVideoSource = () => {
  if (!videoRef.value) return;
  
  const videoUrl = props.video.videoUrl;
  
  // 检查是否是 HLS 格式
  if (videoUrl && videoUrl.includes('.m3u8')) {
    // 使用 HLS.js 播放 HLS 视频
    if (Hls.isSupported()) {
      if (hls.value) {
        hls.value.destroy();
      }
      hls.value = new Hls();
      hls.value.loadSource(videoUrl);
      hls.value.attachMedia(videoRef.value);
      
      hls.value.on(Hls.Events.MANIFEST_PARSED, () => {
        // 视频元数据加载完成后设置倍速和音量
        if (videoRef.value) {
          videoRef.value.playbackRate = currentSpeed.value;
          videoRef.value.volume = volume.value;
          videoRef.value.muted = isMuted.value;
          // 尝试自动播放（可能需要用户交互）
          videoRef.value.play().catch(() => {
            // autoplay 失败时静音播放
            videoRef.value.muted = true;
            isMuted.value = true;
            videoRef.value.play().catch(() => {
              console.log('自动播放失败，需要用户交互');
            });
          });
        }
      });
    } else if (videoRef.value.canPlayType('application/vnd.apple.mpegurl')) {
      // Safari 原生支持 HLS
      videoRef.value.src = videoUrl;
      videoRef.value.playbackRate = currentSpeed.value;
      videoRef.value.volume = volume.value;
      videoRef.value.muted = isMuted.value;
      videoRef.value.play().catch(() => {
        videoRef.value.muted = true;
        isMuted.value = true;
        videoRef.value.play().catch(() => {
          console.log('自动播放失败，需要用户交互');
        });
      });
    }
  } else {
    // 普通视频格式
    videoRef.value.src = videoUrl;
    videoRef.value.playbackRate = currentSpeed.value;
    videoRef.value.volume = volume.value;
    videoRef.value.muted = isMuted.value;
    videoRef.value.play().catch(() => {
      videoRef.value.muted = true;
      isMuted.value = true;
      videoRef.value.play().catch(() => {
        console.log('自动播放失败，需要用户交互');
      });
    });
  }
};

// 视频元信息加载完成时记录总时长
const handleLoadedMetadata = () => {
  if (videoRef.value) {
    duration.value = videoRef.value.duration || 0;
    videoRef.value.playbackRate = currentSpeed.value; // 设置默认倍速
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

// 处理视频加载错误
const handleVideoError = (event) => {
  console.error('视频加载错误:', event);
  // 如果 HLS 加载失败，尝试使用普通方式
  if (hls.value) {
    hls.value.destroy();
    hls.value = null;
  }
  // 可以在这里添加错误提示或备用方案
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
  // 初始化视频源
  initVideoSource();
});

// 点击进度条跳转到对应播放时间
const seekVideo = (event) => {
  if (!videoRef.value) return;
  // 如果 duration 还没有加载，尝试从视频元素获取
  const videoDuration = duration.value || videoRef.value.duration || 0;
  if (!videoDuration) return;
  
  const track = event.currentTarget;
  const rect = track.getBoundingClientRect();
  const clickX = event.clientX - rect.left;
  const percent = Math.min(Math.max(clickX / rect.width, 0), 1);
  const newTime = percent * videoDuration;
  
  // 对于 HLS 视频，使用 seekable 范围
  if (hls.value && videoRef.value.seekable.length > 0) {
    const seekableEnd = videoRef.value.seekable.end(videoRef.value.seekable.length - 1);
    const clampedTime = Math.min(newTime, seekableEnd);
    videoRef.value.currentTime = clampedTime;
    currentTime.value = clampedTime;
  } else {
  videoRef.value.currentTime = newTime;
  currentTime.value = newTime;
  }
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

// 切换倍速
const handleSpeedChange = (speed) => {
  currentSpeed.value = speed;
  if (videoRef.value) {
    videoRef.value.playbackRate = speed;
  }
  showSpeedMenu.value = false;
};

// 处理音量输入
const handleVolumeInput = (event) => {
  const newVolume = parseFloat(event.target.value);
  volume.value = newVolume;
  if (videoRef.value) {
    videoRef.value.volume = newVolume;
    if (newVolume > 0) {
      isMuted.value = false;
      videoRef.value.muted = false;
    }
  }
};

// 切换静音
const toggleMute = () => {
  isMuted.value = !isMuted.value;
  if (videoRef.value) {
    videoRef.value.muted = isMuted.value;
    if (!isMuted.value && volume.value === 0) {
      volume.value = 0.5;
      videoRef.value.volume = 0.5;
    }
  }
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

// 监听视频 URL 变化，重新初始化视频源
watch(
  () => props.video.videoUrl,
  () => {
    initVideoSource();
  },
  { immediate: true }
);

// 监听 isActive 属性变化，控制播放状态
watch(
  () => props.isActive,
  (newVal) => {
    if (videoRef.value) {
      if (newVal) {
        videoRef.value.play().catch(() => {
          // 播放失败时尝试静音播放
          videoRef.value.muted = true;
          videoRef.value.play().catch(() => {
            console.log('播放失败，需要用户交互');
          });
        });
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
  if (hls.value) {
    hls.value.destroy();
  }
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
  overflow: visible;
}

.video-player.is-fullscreen {
  width: 100%;
  height: 100vh;
  padding-right: 0 !important;
}

.video-player.with-comments {
  padding-right: 362px;
}

.video-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: visible;
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
  color: #ffffff;
  font-size: 0.85rem;
  z-index: 15;
}

.progress-track {
  width: 100%;
  height: 2px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  overflow: hidden;
  transition: height 0.2s ease, background 0.2s ease;
  cursor: pointer;
}

.progress-track:hover {
  height: 4px;
  background: rgba(255, 255, 255, 0.6);
}

.progress-fill {
  height: 100%;
  background: rgba(255, 255, 255, 0.5);
  transition: width 0.1s linear;
}

.progress-track:hover .progress-fill {
  background: rgba(255, 255, 255, 0.9);
}

.progress-time {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  font-variant-numeric: tabular-nums;
  font-size:16px;
  gap: 10px;
  flex-wrap: nowrap;
  white-space: nowrap;
}

.time-display {
  white-space: nowrap;
  flex-shrink: 0;
}

.control-bar {
  margin-left: auto;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 12px;
  flex-wrap: nowrap;
  flex-shrink: 0;
}

.control-bar > div {
  white-space: nowrap;
  flex-shrink: 0;
}

.speed-control {
  position: relative;
  cursor: pointer;
}

.speed-menu {
  position: absolute;
  bottom: 12px;
  left: 50%;
  transform: translateX(-50%);
  margin-bottom: 8px;
  background: rgba(0, 0, 0, 0.85);
  backdrop-filter: blur(10px);
  border-radius: 8px;
  padding: 6px 0;
  min-width: 80px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
  z-index: 200;
}

.speed-option {
  padding: 6px 16px;
  color: rgba(255, 255, 255, 0.8);
  cursor: pointer;
  font-size: 0.85rem;
  transition: background-color 0.2s, color 0.2s;
  text-align: center;
}

.speed-option:hover {
  background-color: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.speed-option.active {
  color: #ff2d55;
  font-weight: 600;
}

.volume-control {
  position: relative;
}

.volume-menu {
  position: absolute;
  bottom: 15px;
  left: 50%;
  transform: translateX(-50%);
  margin-bottom: 8px;
  background: rgba(0, 0, 0, 0.85);
  backdrop-filter: blur(10px);
  border-radius: 8px;
  padding: 12px 8px;
  width: 40px;
  height: 120px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
  z-index: 200;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  box-sizing: border-box;
}

.volume-slider-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  flex: 1;
  width: 100%;
  position: relative;
  height: 110px;
}

.volume-value {
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.75rem;
  text-align: center;
  font-weight: 600;
  padding-bottom: 10px;
}

.volume-slider {
  width: 100px;
  height: 4px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 2px;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
  transform: rotate(-90deg);
  transform-origin: center;
  position: absolute;
  top: 50%;
  left: 50%;
  margin-top: 18px;
  margin-left: -50px;
}

.volume-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 10px;
  height: 10px;
  background: #fff;
  border-radius: 50%;
  margin-top: -3px;
}

.volume-slider::-moz-range-thumb {
  width: 16px;
  height: 16px;
  background: #fff;
  border-radius: 50%;
  cursor: pointer;
  border: none;
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


.video-actions ul {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.video-actions li {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #fff;
  cursor: pointer;
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.video-actions ul li i {
  margin: 0;
  padding: 0;
  font-size: 28px;
  display: block;
}

.video-actions li .action-count {
  margin-top: 4px;
  font-size: 14px;
  font-weight: bold;
  text-align: center;
  line-height: 1.2;
  color: #fff;
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