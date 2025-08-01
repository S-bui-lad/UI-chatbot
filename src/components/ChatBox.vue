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
        <div class="bubble" v-if="msg.sender === 'bot'">
          <div v-if="Array.isArray(msg.text)">
            <div
              v-for="(step, stepIndex) in msg.text"
              :key="stepIndex"
              class="register-step"
            >
              <div style="display: flex; align-items: center; gap: 8px; flex-wrap: wrap;">
              <strong>Bước {{ step.step }}:</strong>
              <p style="margin: 0;">{{ step.title }}</p>
            </div>
              <p v-if="step.description">{{ step.description }}</p>
              <img 
                v-if="step.image_url"
                :src="step.image_url"
                alt="step-image"
                class="step-image"
                style="max-width: 100%;
                  max-height: 200px;
                  border-radius: 8px;
                  margin-bottom: 4px;
                  margin-top: 4px;
                  object-fit: contain;"
              />
            </div>
          </div>
          <div v-else v-html="formatBotText(msg.text)" />
        </div>

        <!-- User chat -->
        <div class="bubble" v-else>
          <p v-html="formatBotText(msg.text)"></p>
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
      <textarea
        v-model="userInput"
        @keydown.enter="handleEnter"
        placeholder="Nhập nội dung..."
        rows="1"
      ></textarea>
      <button @click="sendMessage" :disabled="loading">Gửi</button>
    </div>

    <!-- Support Form Modal -->
    <div v-if="showSupportForm" class="support-form-overlay" @click="closeSupportForm">
      <div class="support-form" @click.stop>
        <h3>Thông tin hỗ trợ kỹ thuật</h3>
        
        <form @submit.prevent="handleSupportSubmit">
          <div class="form-group">
            <label>Họ tên *</label>
            <input
              v-model="supportForm.fullName"
              type="text"
              required
            />
          </div>

          <div class="form-group">
            <label>Tên tổ chức</label>
            <input
              v-model="supportForm.organization"
              type="text"
            />
          </div>

          <div class="form-group">
            <label>Số điện thoại</label>
            <input
              v-model="supportForm.phone"
              type="tel"
            />
          </div>

          <div class="form-group">
            <label>Email</label>
            <input
              v-model="supportForm.email"
              type="email"
            />
          </div>

          <div class="form-group">
            <label>Mô tả chi tiết lỗi</label>
            <textarea
              v-model="supportForm.issueDescription"
              rows="4"
            ></textarea>
          </div>

          <div class="form-group">
            <label>Hình ảnh lỗi</label>
            <input
              ref="fileInput"
              type="file"
              multiple
              accept="image/*"
              @change="handleImageUpload"
            />
            <div class="image-preview">
              <div 
                v-for="(image, index) in supportForm.images" 
                :key="index"
                class="image-item"
              >
                <img :src="image.preview" :alt="`Preview ${index}`" />
                <button 
                  type="button" 
                  @click="removeImage(index)"
                  class="remove-image"
                >
                  ×
                </button>
              </div>
            </div>
          </div>

          <div class="form-actions">
            <button type="submit" class="submit-btn" :disabled="isSubmitting">
              {{ isSubmitting ? 'Đang gửi...' : 'Gửi yêu cầu hỗ trợ' }}
            </button>
            <button type="button" @click="closeSupportForm" class="cancel-btn">
              Hủy
            </button>
          </div>
        </form>
      </div>
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
      loading: false,
      // Support form data
      showSupportForm: false,
      isSubmitting: false,
      supportForm: {
        fullName: '',
        organization: '',
        phone: '',
        email: '',
        issueDescription: '',
        images: []
      }
    }
  },
  async mounted() {
    await this.loadChatHistory()
  },
  methods: {
    async loadChatHistory() {
      const user_id = localStorage.getItem('user_id')
      if (!user_id) {
        console.log('Không tìm thấy user_id')
        return
      }

      console.log('Đang tải lịch sử chat cho user_id:', user_id)

      try {
        const response = await axios.get(
          `http://localhost:8000/history/${user_id}`,
          {
            headers: {
              'Authorization': `Bearer ${localStorage.getItem('token')}`,
            }
          }
        )
        
        console.log('Response từ API history:', response.data)
        
        if (response.data && response.data.history && Array.isArray(response.data.history)) {
          let rawMessages = response.data.history.map(item => {
            console.log('Processing item:', item)
            return {
              sender: item.role === 'user' ? 'user' : 'bot',
              text: item.content || item.message || item.text || '',
              user_id: user_id,
              timestamp: new Date(item.timestamp).getTime() 
            }
          })
          
          console.log('Raw messages từ API:', rawMessages)
      
          let userMessages = rawMessages.filter(msg => msg.sender === 'user')
          let assistantMessages = rawMessages.filter(msg => msg.sender === 'bot')
          
          console.log('User messages:', userMessages)
          console.log('Assistant messages:', assistantMessages)
          
          let sortedMessages = []
          
          if (userMessages.length === assistantMessages.length) {
            for (let i = 0; i < userMessages.length; i++) {
              sortedMessages.push(userMessages[i])
              sortedMessages.push(assistantMessages[i])
            }
          } else {
            sortedMessages = [...userMessages, ...assistantMessages]
          }
          
          this.messages = sortedMessages
          console.log('Messages cuối cùng:', this.messages)
          
          this.$nextTick(() => {
            this.scrollToBottom()
          })
        } else if (response.data && Array.isArray(response.data) && response.data.length > 0) {
          let rawMessages = response.data.map(item => {
            console.log('Processing item:', item)
            return {
              sender: item.role === 'user' ? 'user' : 'bot',
              text: item.content || item.message || item.text || '',
              user_id: user_id,
              timestamp: item.timestamp ? new Date(item.timestamp).getTime() : Date.now()
            }
          })
          
          let userMessages = rawMessages.filter(msg => msg.sender === 'user')
          let assistantMessages = rawMessages.filter(msg => msg.sender === 'bot')
          
          let sortedMessages = []
          
          if (userMessages.length === assistantMessages.length) {
            for (let i = 0; i < userMessages.length; i++) {
              sortedMessages.push(userMessages[i])
              sortedMessages.push(assistantMessages[i])
            }
          } else {
            sortedMessages = [...userMessages, ...assistantMessages]
          }
          
          this.messages = sortedMessages
          
          this.$nextTick(() => {
            this.scrollToBottom()
          })
        } else {
          console.log('Không có dữ liệu lịch sử hoặc format không đúng')
        }
      } catch (error) {
        console.error('Lỗi khi tải lịch sử chat:', error)
        console.error('Error details:', error.response?.data)
      }
    },
    async sendMessage() {
      try {
        const message = this.userInput.trim()
        if (!message || this.loading) return
        
        if (this.messages.length >= 100) {
          this.messages.shift() 
        }
        
        const user_id = localStorage.getItem('user_id')
        if (!user_id) {
          this.messages.push({
            sender: 'bot',
            text: '⚠️ Không tìm thấy user_id. Vui lòng đăng nhập lại.'
          })
          return
        }

        // Thêm tin nhắn của user
        this.messages.push({ sender: 'user', text: message, user_id })
        this.userInput = ''
        this.loading = true

        this.$nextTick(() => {
          this.scrollToBottom()
        })

        // Gửi request đến server
        const res = await axios.post(
          'http://localhost:8000/chat',
          {
            user_id,
            message
          },
          {
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${localStorage.getItem('token')}`,
            },
            timeout: 30000 // Thêm timeout 30 giây
          }
        )
        
        // Thêm phản hồi từ bot
        if (res.data && res.data.reply) {
          this.messages.push({ sender: 'bot', text: res.data.reply, user_id })
          
          // Check if this is a support request
          if (res.data.metadata && res.data.metadata.requires_support_form) {
            this.showSupportForm = true
          }
        } else {
          this.messages.push({ 
            sender: 'bot', 
            text: '⚠️ Không nhận được phản hồi từ máy chủ.',
            user_id 
          })
        }
        
      } catch (err) {
        console.error('Lỗi khi gửi tin nhắn:', err)
        
        let errorMessage = '⚠️ Lỗi kết nối máy chủ.'
        
        if (err.response) {
          // Server trả về lỗi
          if (err.response.status === 401) {
            errorMessage = '⚠️ Phiên đăng nhập đã hết hạn. Vui lòng đăng nhập lại.'
          } else if (err.response.status === 500) {
            errorMessage = '⚠️ Lỗi máy chủ. Vui lòng thử lại sau.'
          } else if (err.response.data && err.response.data.message) {
            errorMessage = `⚠️ ${err.response.data.message}`
          }
        } else if (err.request) {
          // Không thể kết nối đến server
          errorMessage = '⚠️ Không thể kết nối đến máy chủ. Vui lòng kiểm tra kết nối mạng.'
        }
        
        this.messages.push({
          sender: 'bot',
          text: errorMessage,
          user_id: localStorage.getItem('user_id')
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
    },
    formatBotText(text) {
      if (typeof text !== 'string') {
        text = String(text); // hoặc JSON.stringify(text) nếu là object
      }
      return text.replace(/\n/g, '<br/>');
    },
    handleEnter(event) {
      if (event.shiftKey) {
        return; // Cho phép xuống dòng nếu Shift+Enter
      }
      event.preventDefault(); // Ngăn xuống dòng mặc định
      this.sendMessage().catch(err => {
        console.error('Lỗi khi gửi tin nhắn từ handleEnter:', err)
      })
    },

    // Support form methods
    handleImageUpload(event) {
      const files = Array.from(event.target.files)
      
      files.forEach(file => {
        const reader = new FileReader()
        reader.onload = (e) => {
          this.supportForm.images.push({
            file: file,
            preview: e.target.result
          })
        }
        reader.readAsDataURL(file)
      })
    },

    removeImage(index) {
      this.supportForm.images.splice(index, 1)
    },

    closeSupportForm() {
      this.showSupportForm = false
      // Reset form
      this.supportForm = {
        fullName: '',
        organization: '',
        phone: '',
        email: '',
        issueDescription: '',
        images: []
      }
      if (this.$refs.fileInput) {
        this.$refs.fileInput.value = ''
      }
    },

    async handleSupportSubmit() {
      if (!this.supportForm.fullName.trim()) {
        alert('Vui lòng nhập họ tên')
        return
      }

      this.isSubmitting = true
      
      try {
        const formData = new FormData()
        const user_id = localStorage.getItem('user_id')
        formData.append('user_id', user_id)
        formData.append('full_name', this.supportForm.fullName)
        formData.append('organization', this.supportForm.organization)
        formData.append('phone', this.supportForm.phone)
        formData.append('email', this.supportForm.email)
        formData.append('issue_description', this.supportForm.issueDescription)
        
        this.supportForm.images.forEach((image, index) => {
          formData.append(`image_${index}`, image.file)
        })

        const response = await axios.post(
          'http://localhost:8000/support-request',
          formData,
          {
            headers: {
              'Authorization': `Bearer ${localStorage.getItem('token')}`,
              'Content-Type': 'multipart/form-data'
            }
          }
        )
        
        const result = response.data
        
        if (result.success) {
          alert(result.message)
          this.closeSupportForm()
          
          // Add success message to chat
          this.messages.push({
            sender: 'bot',
            text: `✅ ${result.message}`,
            user_id: user_id
          })
          
          this.$nextTick(() => {
            this.scrollToBottom()
          })
        } else {
          alert(result.message || 'Có lỗi xảy ra, vui lòng thử lại')
        }
      } catch (error) {
        console.error('Error submitting support request:', error)
        alert('Có lỗi xảy ra, vui lòng thử lại')
      } finally {
        this.isSubmitting = false
      }
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

.chat-bubble .bubble p {
  margin: 0.1rem 0;
}

.chat-bubble .bubble br {
  margin-bottom: 0.1rem;
  display: block;
  content: "";
}

.chat-bubble .bubble img {
  margin: 0.25rem 0;
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
  align-items: justify;
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

.chat-input textarea {
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

/* Support Form Styles */
.support-form-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.support-form {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  max-width: 500px;
  width: 90%;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.support-form h3 {
  margin: 0 0 1.5rem 0;
  color: #333;
  font-size: 1.25rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
  color: #555;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 0.9rem;
  transition: border-color 0.2s;
  box-sizing: border-box;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #42b983;
  box-shadow: 0 0 0 2px rgba(66, 185, 131, 0.25);
}

.image-preview {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.5rem;
  flex-wrap: wrap;
}

.image-item {
  position: relative;
  display: inline-block;
}

.image-item img {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 8px;
  border: 1px solid #ddd;
}

.remove-image {
  position: absolute;
  top: -5px;
  right: -5px;
  background: #dc3545;
  color: white;
  border: none;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  font-size: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.form-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1.5rem;
}

.submit-btn,
.cancel-btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s;
}

.submit-btn {
  background: #42b983;
  color: white;
  flex: 1;
}

.submit-btn:hover:not(:disabled) {
  background: #36976f;
}

.submit-btn:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

.cancel-btn {
  background: #6c757d;
  color: white;
}

.cancel-btn:hover {
  background: #545b62;
}

/* Responsive */
@media (max-width: 768px) {
  .support-form {
    width: 95%;
    padding: 1.5rem;
  }
  
  .form-actions {
    flex-direction: column;
  }
}
</style>