<!-- src/components/CommentPanel.vue -->
<template>
  <!-- 评论区主体 -->
  <div class="comment-panel" :class="{ show: isShow }">
    <!-- 头部：标题 + 关闭按钮 -->
    <div class="comment-header-bg">
      <ul>
        <li 
          :class="{ active: activeTab === 0 }" 
          @click="handleTabClick(0)"
        >TA的作品</li>
        <li 
          :class="{ active: activeTab === 1 }" 
          @click="handleTabClick(1)"
        >评论</li>
        <li 
          :class="{ active: activeTab === 2 }" 
          @click="handleTabClick(2)"
        >相关推荐</li>
        <li><button class="close-btn" @click="handleClose">×</button></li>
      </ul>
    </div>
    <div class="comment-header">
      <h3>全部评论 ({{ commentList.length }})</h3>
    </div>

    <!-- 评论列表 -->
    <div class="comment-list">
      <div class="comment-item" v-for="(comment, idx) in commentList" :key="idx">
        <img :src="comment.avatar" alt="Comment Avatar" class="comment-avatar" />
        <div class="comment-content">
          <div class="comment-author">{{ comment.author }}</div>
          <div class="comment-text">{{ comment.content }}</div>
                    <!-- 评论元信息：时间 + 互动按钮 -->
          <div class="comment-meta">
            <span class="comment-time">{{ comment.time }}</span>
            <!-- 新增：3个互动按钮 -->
            <div class="comment-actions">
              <!-- 回复按钮 -->
              <button class="action-btn reply-btn" @click="handleReply(comment, idx)">
                <i class="iconfont icon-huifu"></i>回复
              </button>
              <!-- 分享按钮 -->
              <button class="action-btn share-btn" @click="handleShare(comment)">
                <i class="iconfont icon-huifu1"></i>分享
              </button>
              <!-- 点赞按钮 -->
              <button class="action-btn like-btn" @click="handleCommentLike(idx)">
                <i class="iconfont icon-aixin" :class="{ liked: comment.isLiked }"></i> {{ comment.likeCount || 0 }}
              </button>
            </div>
          </div>
          <!-- 回复列表 -->
          <div class="replies-list" v-if="comment.replies && comment.replies.length > 0 && comment.isShowReplies">
            <div class="reply-item" v-for="(reply, replyIdx) in comment.replies" :key="replyIdx">
              <img :src="reply.avatar" alt="Reply Avatar" class="reply-avatar" />
              <div class="reply-content">
                <div class="reply-author">{{ reply.author }}</div>
                <div class="reply-text">{{ reply.content }}</div>
                <div class="reply-meta">
                  <span class="reply-time">{{ reply.time }}</span>
                </div>
              </div>
            </div>
          </div>
          <!-- 展开/收起回复按钮（在最下方单独显示） -->
          <div class="expand-replies" v-if="comment.replyCount > 0">
            <button 
              class="expand-reply-btn" 
              @click="handleToggleReplies(idx)"
            >
              <span v-if="!comment.isShowReplies">展开{{ comment.replyCount }}条回复</span>
              <span v-else>收起回复</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 输入区域 -->
    <div class="comment-input-area">
      <input
        type="text"
        v-model="newCommentContent"
        placeholder="说点什么..."
        @keyup.enter="handleSubmit"
      />
      <button class="submit-btn" @click="handleSubmit">发送</button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

// 接收父组件传递的参数
const props = defineProps({
  isShow: { // 控制评论区显示/隐藏
    type: Boolean,
    default: false
  },
  videoId: { // 关联的视频ID（后续对接后端用）
    type: [Number, String],
    required: true
  },
  initComments: { // 初始评论列表
    type: Array,
    default: () => []
  }
});

// 向父组件触发事件
const emit = defineEmits(['close', 'submitComment']);

// 响应式变量
const newCommentContent = ref(""); // 新评论输入内容
const commentList = ref(props.initComments); // 评论列表（基于父组件传递的初始数据）
const activeTab = ref(1); // 添加激活的 tab 索引，默认激活第二个（"评论"）

// ... 现有的 watch 和函数 ...

// 添加切换 tab 的函数
const handleTabClick = (index) => {
  activeTab.value = index;
};

// 展开/收起回复
const handleToggleReplies = (idx) => {
  if (!commentList.value[idx].isShowReplies) {
    // 如果回复数据不存在，初始化回复数据
    if (!commentList.value[idx].replies && commentList.value[idx].replyCount > 0) {
      commentList.value[idx].replies = generateMockReplies(commentList.value[idx].replyCount);
    }
    commentList.value[idx].isShowReplies = true;
  } else {
    commentList.value[idx].isShowReplies = false;
  }
};

// 生成模拟回复数据
const generateMockReplies = (count) => {
  return Array.from({ length: count }, (_, i) => ({
    avatar: `https://picsum.photos/id/${100 + i}/100/100`,
    author: `回复用户${String.fromCharCode(65 + i)}`,
    content: `这是第${i + 1}条回复内容，回复的内容会显示在这里。`,
    time: `${i + 1}小时前`
  }));
};

// 回复评论
const handleReply = (comment, idx) => {
  // TODO: 实现回复功能
  console.log('回复评论', comment, idx);
};

// 分享评论
const handleShare = (comment) => {
  // TODO: 实现分享功能
  console.log('分享评论', comment);
};

// 点赞评论
const handleCommentLike = (idx) => {
  if (!commentList.value[idx].isLiked) {
    commentList.value[idx].isLiked = true;
    commentList.value[idx].likeCount = (commentList.value[idx].likeCount || 0) + 1;
  } else {
    commentList.value[idx].isLiked = false;
    commentList.value[idx].likeCount = Math.max((commentList.value[idx].likeCount || 0) - 1, 0);
  }
};

// 监听父组件传递的 initComments 变化（比如切换视频时更新评论）
watch(
  () => props.initComments,
  (newVal) => {
    // 初始化评论数据，确保有 isShowReplies 属性
    commentList.value = newVal.map(comment => ({
      ...comment,
      isShowReplies: comment.isShowReplies || false,
      isLiked: comment.isLiked || false,
      likeCount: comment.likeCount || 0
    }));
  },
  { deep: true, immediate: true }
);

// 关闭评论区：通知父组件更新 isShow 状态
const handleClose = () => {
  emit('close');
  newCommentContent.value = ""; // 清空输入框
};

// 提交评论：通知父组件（后续父组件可对接后端）
const handleSubmit = () => {
  const content = newCommentContent.value.trim();
  if (!content) return;

  // 构造新评论数据（前端临时模拟，后续由后端返回）
  const newComment = {
    avatar: "https://picsum.photos/id/99/100/100", // 当前用户头像
    author: "我",
    content: content,
    time: "刚刚"
  };

  // 通知父组件提交评论（父组件可选择是否调用后端接口）
  emit('submitComment', newComment);

  // 前端临时更新列表（实际项目中应等待后端响应后再更新）
  commentList.value.unshift(newComment);
  newCommentContent.value = "";
};
</script>

<style scoped>

.comment-header-bg ul{
  display: flex;
  justify-content: space-around;
  align-items: center;
}
.comment-header-bg ul  li{
  padding: 10px 0;
}
.comment-header-bg ul li:hover {
  border-bottom: 2px solid #ff0000;
}
.comment-header-bg ul li.active {
  border-bottom: 2px solid #ff0000;
}
.comment-header-bg ul li {
  padding: 10px 0;
  cursor: pointer;
  position: relative;
  border-bottom: 2px solid transparent; /* 默认透明边框，保持布局稳定 */
  transition: border-bottom-color 0.2s ease;
}

.comment-header-bg ul li:hover {
  border-bottom-color: rgba(255, 0, 0, 0.5); /* hover 时半透明红色 */
}

.comment-header-bg ul li.active {
  border-bottom: 2px solid #ff0000; /* 激活时红色底部边框 */
}

/* 关闭按钮不需要这些样式 */
.comment-header-bg ul li:last-child {
  border-bottom: none;
  cursor: default;
}

.comment-header-bg ul li:last-child:hover {
  border-bottom: none;
}
/* 评论区主体 */
.comment-panel {
  position: fixed;
  top: 0;
  right: 2px;
  width: 360px;
  height: 85%;
  background-color: #111217;
  color: #fff;
  z-index: 100;
  transform: translateX(100%);
  transition: transform 0.3s ease;
  display: flex;
  flex-direction: column;
  box-shadow: 0 0 25px rgba(0, 0, 0, 0.45);
}
.comment-panel.show {
  transform: translateX(0);
}

/* 头部 */
.comment-header {
  padding: 16px 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  display: flex;
  justify-content: space-between;
  align-items: center;

}
.comment-header h3 {
  font-size: 12px;

}
.close-btn {
  background: transparent;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: rgba(255, 255, 255, 0.6);
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 评论列表 */
.comment-list {
  flex: 1;
  overflow-y: auto;
  padding: 16px 20px;
}
.comment-item {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
.comment-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  object-fit: cover;
}
.comment-content {
  flex: 1;
}
.comment-author {
  font-size: 0.9rem;
  font-weight: 600;
  color: #fff;
  margin-bottom: 4px;
}
.comment-text {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.75);
  margin-bottom: 4px;
}
.comment-time {
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.45);
}

/* 输入区域 */
.comment-input-area {
  padding: 16px 20px 18px;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  display: flex;
  gap: 10px;
}
.comment-input-area input {
  flex: 1;
  padding: 10px 16px;
  border: none;
  border-radius: 24px;
  outline: none;
  font-size: 0.9rem;
  background: rgba(255, 255, 255, 0.12);
  color: #fff;
}
.submit-btn {
  padding: 10px 20px;
  background-color: #ff2d55;
  color: white;
  border: none;
  border-radius: 24px;
  cursor: pointer;
  font-size: 0.9rem;
}
.submit-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .comment-panel {
    width: 100%;
  }
}
/* 评论元信息（时间 + 互动按钮） */
.comment-meta {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: 8px;
}

.comment-time {
  font-size: 0.7rem;
  color: #999;
}

/* 新增：评论互动按钮容器 */
.comment-actions {
  display: flex;
  gap: 15px;
  font-size: 0.8rem;
}

/* 单个互动按钮样式 */
.action-btn {
  background: transparent;
  border: none;
  color: #999;
  cursor: pointer;
  padding: 2px 0;
  display: flex;
  align-items: center;
  gap: 4px;
  transition: color 0.2s;
}

/* 按钮hover效果 */
.action-btn:hover {
  color: #ffffff;
}

/* 点赞图标样式 */
.icon-aixin.liked {
  color: #ff3b30;
}


/* 回复列表样式 */
.replies-list {
  margin-top: 12px;
}

.reply-item {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}

.reply-avatar {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
}

.reply-content {
  flex: 1;
}

.reply-author {
  font-size: 0.85rem;
  font-weight: 600;
  color: #fff;
  margin-bottom: 4px;
}

.reply-text {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.75);
  margin-bottom: 4px;
  line-height: 1.4;
}

.reply-meta {
  display: flex;
  align-items: center;
  gap: 8px;
}

.reply-time {
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.45);
}

/* 展开/收起回复按钮 */
.expand-replies {
  margin-top: 8px;
}

.expand-reply-btn {
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  padding: 4px 0;
  font-size: 0.8rem;
  transition: color 0.2s;
  display: flex;
  align-items: center;
}

.expand-reply-btn:hover {
  color: rgba(255, 255, 255, 0.9);
}

.reply-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.7rem;
  color: #999;
}

/* 回复点赞按钮 */
.reply-like-btn {
  background: transparent;
  border: none;
  color: #999;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 4px;
  transition: color 0.2s;
}

.reply-like-btn:hover {
  color: #ff3b30;
}

/* 新增：回复输入框区域 */
.reply-input-area {
  margin-top: 12px;
  display: flex;
  gap: 8px;
  align-items: center;
}

.reply-input-area input {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #e0e0e0;
  border-radius: 16px;
  outline: none;
  font-size: 0.8rem;
}

.submit-reply-btn {
  padding: 6px 12px;
  background-color: #007aff;
  color: white;
  border: none;
  border-radius: 16px;
  cursor: pointer;
  font-size: 0.8rem;
  transition: background-color 0.2s;
}

.submit-reply-btn:hover {
  background-color: #0066cc;
}

/* 移动端适配：调整按钮间距和回复列表缩进 */
@media (max-width: 768px) {
  .comment-actions {
    gap: 10px;
  }

  .reply-list {
    margin-left: 36px;
  }
}
</style>