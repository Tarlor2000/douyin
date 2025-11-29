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
          <div class="comment-time">{{ comment.time }}</div>
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
// 监听父组件传递的 initComments 变化（比如切换视频时更新评论）
watch(
  () => props.initComments,
  (newVal) => {
    commentList.value = newVal;
  },
  { deep: true }
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
  right: 0;
  width: 360px;
  height: 100%;
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
</style>