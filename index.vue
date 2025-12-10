<template>
  <div class="chat-container">
    <div class="message-list" ref="messageListRef">
      <div v-for="(message, index) in messages" :key="index"
        :class="['message-row', message.sender === 'user' ? 'user-message' : 'ai-message']">
        <img :src="message.sender === 'user' ? userAvatar : aiAvatar" class="avatar" alt="Avatar" />
        <div class="message-bubble">
          {{ message.content }}
        </div>
      </div>

      <div v-if="isLoading" class="message-row ai-message">
        <img :src="aiAvatar" class="avatar" alt="AI Avatar" />
        <div class="message-bubble loading-dots">
          <span>.</span><span>.</span><span>.</span>
        </div>
      </div>
    </div>

    <div class="input-area">
      <textarea v-model="currentInput" @keydown.enter.prevent="sendMessage" placeholder="请输入" rows="1"></textarea>
      <button @click="sendMessage" :disabled="!currentInput.trim() || !canInput">
        <svg fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 10l7-7m0 0l7 7m-7-7v18"
            transform="rotate(90 12 12)" />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, watch } from 'vue';

// --- 状态定义 ---
// 消息列表，包含发送者（'user' 或 'ai'）和内容
const messages = ref([
  // 初始消息示例 (可选)
  { sender: 'ai', content: '您好，有什么我可以帮您的吗？' }
]);
const currentInput = ref('');
const isLoading = ref(false);
const canInput = ref(true)

// 头像图片路径 (请替换成您自己的图片路径)
const userAvatar = 'path/to/user-avatar.png'; // 替换为用户头像
const aiAvatar = 'path/to/ai-avatar.png';   // 替换为AI头像

// DOM引用，用于滚动到底部
const messageListRef = ref(null);

// --- 方法 ---

/**
 * 滚动消息列表到最新消息
 */
const scrollToBottom = () => {
  nextTick(() => {
    const list = messageListRef.value;
    if (list) {
      list.scrollTop = list.scrollHeight;
    }
  });
};

// 监听消息变化，自动滚动
watch(messages, scrollToBottom, { deep: true });

/**
 * 处理用户发送消息
 */
const sendMessage = () => {
  const content = currentInput.value.trim();
  if (!content || isLoading.value) return;

  // 1. 添加用户消息
  messages.value.push({ sender: 'user', content });
  currentInput.value = '';
  isLoading.value = true;
  canInput.value = false
  // 模拟AI回复
  simulateAiResponse(content);
};

/**
 * 模拟AI的异步回复过程
 * @param {string} userMessage 用户发送的消息内容
 */
const simulateAiResponse = async (userMessage) => {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 5000); // 5 秒超时
  try {
    const response = await fetch("http://localhost:8080/chat", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ question: userMessage })
    });

    isLoading.value = false;
    const reader = response.body.getReader();
    const decoder = new TextDecoder();

    let aiMessage = ""

    messages.value.push({ sender: 'ai', content: aiMessage })
    while (true) {

      const { done, value } = await reader.read();

      if (done) { canInput.value = true; break };

      const chunk = decoder.decode(value);

      messages.value[messages.value.length - 1].content += chunk
    }
  }
  catch (err) {
    isLoading.value = false
    canInput.value = true
    messages.value.push({ sender: 'ai', content: "请求超时或被中断,请检查网络" });
    console.error("请求超时或被中断:", err);
  } finally {
    clearTimeout(timeout);
  }
}
</script>

<style scoped>
/* --- 布局样式 --- */
.chat-container {
  display: flex;
  flex-direction: column;
  width: 100%;
  /* 限制聊天框宽度 */
  height: 500px;
  /* 限制聊天框高度 */
  margin: 20px auto;
  border: 1px solid #e0e0e0;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  background-color: #fff;
  overflow: hidden;
}

/* 消息列表区域 */
.message-list {
  flex-grow: 1;
  padding: 15px;
  overflow-y: auto;
  background-color: #f7f7f7;
  scroll-behavior: smooth;
}

/* 输入区域 */
.input-area {
  display: flex;
  padding: 10px 15px;
  border-top: 1px solid #eee;
  background-color: #fff;
}

/* --- 消息行样式 --- */
.message-row {
  display: flex;
  margin-bottom: 15px;
  align-items: flex-start;
}

.avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  margin-top: 5px;
  /* 使头像与气泡顶部对齐 */
  object-fit: cover;
  flex-shrink: 0;
}

.message-bubble {
  max-width: 70%;
  padding: 10px 14px;
  border-radius: 18px;
  font-size: 14px;
  line-height: 1.5;
  word-wrap: break-word;
  word-break: break-all;
}

/* 用户消息 (右侧对齐) */
.user-message {
  justify-content: flex-end;
}

.user-message .avatar {
  order: 2;
  /* 放在气泡右边 */
  margin-left: 10px;
}

.user-message .message-bubble {
  background-color: #409eff;
  /* 蓝色 */
  color: white;
  border-bottom-right-radius: 4px;
  order: 1;
}

/* AI消息 (左侧对齐) */
.ai-message {
  justify-content: flex-start;
}

.ai-message .avatar {
  order: 1;
  /* 放在气泡左边 */
  margin-right: 10px;
}

.ai-message .message-bubble {
  background-color: #fff;
  /* 白色/浅灰色 */
  color: #333;
  border: 1px solid #ddd;
  border-bottom-left-radius: 4px;
  order: 2;
}

/* --- 输入框样式 --- */
.input-area textarea {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  resize: none;
  /* 禁用手动缩放 */
  font-size: 14px;
  line-height: 1.4;
  outline: none;
  transition: border-color 0.3s;
  overflow: hidden;
  /* 隐藏滚动条 */
}

.input-area textarea:focus {
  border-color: #409eff;
}

/* 发送按钮 */
.input-area button {
  width: 40px;
  height: 40px;
  margin-left: 10px;
  background-color: #409eff;
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.3s, opacity 0.3s;
  flex-shrink: 0;
}

.input-area button:disabled {
  background-color: #a0cfff;
  cursor: not-allowed;
}

.input-area button:hover:not(:disabled) {
  background-color: #66b1ff;
}

.input-area button svg {
  width: 20px;
  height: 20px;
  transform: translateX(1px);
  /* 稍微右移，看起来更像发送图标 */
}

/* --- 加载指示器样式 --- */
.loading-dots {
  display: flex;
  align-items: flex-end;
}

.loading-dots span {
  animation: dot-flicker 1.4s infinite ease-in-out both;
  font-size: 20px;
  line-height: 1;
  display: inline-block;
  margin: 0 -2px;
}

.loading-dots span:nth-child(1) {
  animation-delay: 0s;
}

.loading-dots span:nth-child(2) {
  animation-delay: 0.2s;
}

.loading-dots span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes dot-flicker {

  0%,
  80%,
  100% {
    opacity: 0;
  }

  40% {
    opacity: 1;
  }
}
</style>
