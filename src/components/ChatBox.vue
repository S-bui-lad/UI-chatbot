<template>
  <div class="chat-container">
    <div class="chat-header">💬 Hỗ trợ khách hàng AI</div>

    <div class="chat-body" ref="chatBody">
      <div
        v-for="(msg, index) in messages"
        :key="index"
        :class="['chat-bubble', msg.sender]"
      >
        <div class="avatar">
          <img
            v-if="msg.sender === 'bot'"
            src="https://cdn-icons-png.flaticon.com/512/4712/4712109.png"
            alt="bot"
          />
          <img
            v-else
            src="https://cdn-icons-png.flaticon.com/512/1946/1946429.png"
            alt="user"
          />
        </div>
        <div class="bubble">
          <p>{{ msg.text }}</p>
        </div>
      </div>

      <div v-if="loading" class="chat-bubble bot loading">
        <div class="avatar">
          <img src="https://cdn-icons-png.flaticon.com/512/4712/4712109.png" alt="bot" />
        </div>
        <div class="bubble typing">
          <span></span><span></span><span></span>
        </div>
      </div>
    </div>

    <div class="chat-input">
      <input
        v-model="userInput"
        @keyup.enter="sendMessage"
        placeholder="Nhập nội dung..."
      />
      <button @click="sendMessage" :disabled="loading">Gửi</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      userInput: '',
      messages: [],
      loading: false
    }
  },
  methods: {
    async sendMessage() {
      const message = this.userInput.trim()
      if (!message || this.loading) return

      this.messages.push({ sender: 'user', text: message })
      this.userInput = ''
      this.loading = true

      this.$nextTick(() => {
        this.scrollToBottom()
      })

      try {
        const res = await axios.post('http://localhost:8000/chat', {
          message
        })
        this.messages.push({ sender: 'bot', text: res.data.reply })
      } catch (err) {
        this.messages.push({
          sender: 'bot',
          text: '⚠️ Lỗi kết nối máy chủ.'
        })
      } finally {
        this.loading = false
        this.$nextTick(() => {
          this.scrollToBottom()
        })
      }
    },
    scrollToBottom() {
      const el = this.$refs.chatBody
      el.scrollTop = el.scrollHeight
    }
  }
}
</script>

<style scoped>
.chat-container {
  width: 100%;
  max-width: 700px;
  margin: 2rem auto;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  height: 80vh;
}

.chat-header {
  padding: 1rem;
  font-size: 1.25rem;
  font-weight: bold;
  background: #42b983;
  color: white;
  border-top-left-radius: 12px;
  border-top-right-radius: 12px;
  text-align: center;
}

.chat-body {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
  background: #f6f8fa;
}

.chat-bubble {
  display: flex;
  margin-bottom: 1rem;
  align-items: flex-end;
}

.chat-bubble.user {
  justify-content: flex-end;
}

.chat-bubble.bot {
  justify-content: flex-start;
}

.chat-bubble .avatar {
  width: 40px;
  height: 40px;
  margin: 0 0.5rem;
}

.chat-bubble .avatar img {
  width: 100%;
  border-radius: 50%;
}

.chat-bubble .bubble {
  max-width: 65%;
  background: #e4e6eb;
  padding: 0.75rem;
  border-radius: 16px;
  font-size: 0.95rem;
  line-height: 1.4;
}

.chat-bubble.user .bubble {
  background: #42b983;
  color: white;
  border-bottom-right-radius: 0;
}

.chat-bubble.bot .bubble {
  background: #e4e6eb;
  color: #333;
  border-bottom-left-radius: 0;
}

/* Typing dots */
.typing {
  display: flex;
  gap: 4px;
  align-items: center;
  height: 1.2rem;
}
.typing span {
  width: 6px;
  height: 6px;
  background: #999;
  border-radius: 50%;
  animation: blink 1.2s infinite;
}
.typing span:nth-child(2) {
  animation-delay: 0.2s;
}
.typing span:nth-child(3) {
  animation-delay: 0.4s;
}
@keyframes blink {
  0%, 80%, 100% {
    transform: scale(1);
    opacity: 0.3;
  }
  40% {
    transform: scale(1.5);
    opacity: 1;
  }
}

.chat-input {
  display: flex;
  padding: 1rem;
  background: white;
  border-top: 1px solid #eee;
}

.chat-input input {
  flex: 1;
  padding: 0.75rem 1rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  margin-right: 0.5rem;
}

.chat-input button {
  padding: 0.75rem 1.2rem;
  background: #42b983;
  border: none;
  color: white;
  border-radius: 8px;
  cursor: pointer;
}

.chat-input button:hover {
  background: #36976f;
}
</style>
