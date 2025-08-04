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
                  margin: 4px auto;
                  object-fit: contain;
                  display: block;"
              />
            </div>
          </div>
          <div v-else v-html="formatBotText(msg.text)" />
          
          <!-- Support form button -->
          <div v-if="msg.requiresSupportForm" class="support-form-button">
            <button @click="showSupportFormModal" class="support-btn">
              <span class="btn-icon">📋</span>
              <span class="btn-text">Điền thông tin hỗ trợ</span>
              <span class="btn-arrow">→</span>
            </button>
          </div>
          
          <!-- Edit support request button -->
          <div v-if="isSupportRequestSuccess(msg.text)" class="edit-support-button">
            <button @click="showEditSupportForm(msg)" class="edit-support-btn">
              <span class="btn-icon">✏️</span>
              <span class="btn-text">Sửa đổi thông tin</span>
              <span class="btn-arrow">→</span>
            </button>
          </div>
        </div>

        <!-- User chat -->
        <div class="bubble" v-else>
          <p v-html="formatBotText(msg.text)"></p>
          
          <!-- Selected images preview -->
          <div v-if="msg.selectedImages && msg.selectedImages.length > 0" class="message-images">
            <div 
              v-for="(image, index) in msg.selectedImages" 
              :key="index"
              class="message-image-item"
            >
              <img :src="image.preview" :alt="image.name" />
            </div>
          </div>
          
          <!-- Uploaded images -->
          <div v-if="msg.uploadedImages && msg.uploadedImages.length > 0" class="uploaded-images">
            <div 
              v-for="(image, index) in msg.uploadedImages" 
              :key="index"
              class="uploaded-image-item"
            >
              <img :src="image.url" :alt="image.name" />
              <span class="image-name">{{ image.name }}</span>
            </div>
          </div>
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
      <div class="input-container">
        <textarea
          v-model="userInput"
          @keydown.enter="handleEnter"
          placeholder="Nhập nội dung..."
          rows="1"
        ></textarea>
      </div>
      <button @click="sendMessage" :disabled="loading || uploadingImage">
        {{ uploadingImage ? '⏳' : 'Gửi' }}
      </button>
    </div>

    <!-- Support Form Modal -->
    <div v-if="showSupportForm" class="support-form-overlay" @click="closeSupportForm">
      <div class="support-form" @click.stop>
        <h3>{{ currentRequestId ? 'Sửa đổi thông tin hỗ trợ' : 'Thông tin hỗ trợ kỹ thuật' }}</h3>
        
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
              {{ isSubmitting ? 'Đang gửi...' : (currentRequestId ? 'Cập nhật yêu cầu hỗ trợ' : 'Gửi yêu cầu hỗ trợ') }}
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
      },
      // Image upload data
      uploadingImage: false,
      // Chat image data
      selectedImages: [],
      // Current request ID for editing
      currentRequestId: null
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

        // Thêm tin nhắn của user với preview ảnh
        const userMessage = { 
          sender: 'user', 
          text: message, 
          user_id,
          selectedImages: [...this.selectedImages] // Copy selected images
        }
        this.messages.push(userMessage)
        
        // Clear input và selected images
        this.userInput = ''
        this.selectedImages = []
        this.loading = true

        this.$nextTick(() => {
          this.scrollToBottom()
        })

        // Upload ảnh nếu có
        let uploadedImages = []
        if (userMessage.selectedImages && userMessage.selectedImages.length > 0) {
          this.uploadingImage = true
          try {
            for (const imageData of userMessage.selectedImages) {
              const formData = new FormData()
              formData.append('image', imageData.file)
              formData.append('user_id', user_id)

              const response = await axios.post(
                'http://localhost:8000/upload-image',
                formData,
                {
                  headers: {
                    'Authorization': `Bearer ${localStorage.getItem('token')}`,
                    'Content-Type': 'multipart/form-data'
                  }
                }
              )

              if (response.data && response.data.success) {
                uploadedImages.push({
                  name: imageData.name,
                  url: response.data.image_url || response.data.url
                })
              }
            }
          } catch (error) {
            console.error('Lỗi khi upload ảnh:', error)
            // Vẫn tiếp tục gửi tin nhắn dù upload ảnh thất bại
          } finally {
            this.uploadingImage = false
          }
        }

        // Cập nhật tin nhắn với thông tin ảnh đã upload
        const lastMessage = this.messages[this.messages.length - 1]
        if (lastMessage && lastMessage.sender === 'user') {
          lastMessage.uploadedImages = uploadedImages
          if (uploadedImages.length > 0) {
            lastMessage.text += `\n📷 Đã upload ${uploadedImages.length} ảnh: ${uploadedImages.map(img => img.name).join(', ')}`
          }
        }

        // Gửi request đến server với thông tin ảnh
        const chatData = {
          user_id,
          message: message
        }
        
        if (uploadedImages.length > 0) {
          chatData.images = uploadedImages
        }

        const res = await axios.post(
          'http://localhost:8000/chat',
          chatData,
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
          const botMessage = { 
            sender: 'bot', 
            text: res.data.reply, 
            user_id,
            requiresSupportForm: false
          }
          
          // Check if this is a support request
          if (res.data.metadata && res.data.metadata.requires_support_form) {
            botMessage.requiresSupportForm = true
          }
          
          this.messages.push(botMessage)
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
    showSupportFormModal() {
      this.showSupportForm = true
    },
    
    async handleImageUpload(event) {
      const files = Array.from(event.target.files)
      const user_id = localStorage.getItem('user_id')
      
      for (const file of files) {
        // Kiểm tra kích thước file (giới hạn 5MB)
        if (file.size > 5 * 1024 * 1024) {
          alert(`File ${file.name} quá lớn. Vui lòng chọn file nhỏ hơn 5MB.`)
          continue
        }

        // Kiểm tra loại file
        if (!file.type.startsWith('image/')) {
          alert(`File ${file.name} không phải là ảnh hợp lệ.`)
          continue
        }

        try {
          // Upload ảnh lên server
          const formData = new FormData()
          formData.append('image', file)
          formData.append('user_id', user_id)

          const response = await axios.post(
            'http://localhost:8000/upload-image',
            formData,
            {
              headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`,
                'Content-Type': 'multipart/form-data'
              }
            }
          )

          if (response.data && response.data.success) {
            // Tạo preview và lưu thông tin ảnh đã upload
            const reader = new FileReader()
            reader.onload = (e) => {
              this.supportForm.images.push({
                file: file,
                preview: e.target.result,
                name: file.name,
                url: response.data.image_url || response.data.url // URL từ server
              })
            }
            reader.readAsDataURL(file)
          } else {
            alert(`Lỗi upload ảnh ${file.name}`)
          }
        } catch (error) {
          console.error('Lỗi upload ảnh:', error)
          alert(`Lỗi upload ảnh ${file.name}`)
        }
      }
    },

    // Chat image selection method
    handleChatImageSelect(event) {
      const files = Array.from(event.target.files)
      
      files.forEach(file => {
        // Kiểm tra kích thước file (giới hạn 5MB)
        if (file.size > 5 * 1024 * 1024) {
          alert(`File ${file.name} quá lớn. Vui lòng chọn file nhỏ hơn 5MB.`)
          return
        }

        // Kiểm tra loại file
        if (!file.type.startsWith('image/')) {
          alert(`File ${file.name} không phải là ảnh hợp lệ.`)
          return
        }

        // Tạo preview cho ảnh
        const reader = new FileReader()
        reader.onload = (e) => {
          this.selectedImages.push({
            file: file,
            preview: e.target.result,
            name: file.name
          })
        }
        reader.readAsDataURL(file)
      })

      // Reset input file
      if (this.$refs.imageInput) {
        this.$refs.imageInput.value = ''
      }
    },

    // Remove selected image
    removeSelectedImage(index) {
      this.selectedImages.splice(index, 1)
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
      // Reset current request ID
      this.currentRequestId = null
      if (this.$refs.fileInput) {
        this.$refs.fileInput.value = ''
      }
    },



    // Phát hiện tin nhắn yêu cầu hỗ trợ đã thành công
    isSupportRequestSuccess(text) {
      if (typeof text !== 'string') return false
      return text.includes('✅ YÊU CẦU HỖ TRỢ ĐÃ ĐƯỢC GỬI THÀNH CÔNG') ||
             text.includes('Mã yêu cầu') ||
             text.includes('Trạng thái')
    },

    // Hiển thị form sửa đổi thông tin
    showEditSupportForm(msg) {
      // Parse thông tin từ tin nhắn
      const text = msg.text
      const info = this.parseSupportRequestInfo(text)
      
      // Điền thông tin vào form
      this.supportForm = {
        fullName: info.fullName || '',
        organization: info.organization || '',
        phone: info.phone || '',
        email: info.email || '',
        issueDescription: info.issueDescription || '',
        images: []
      }
      
      // Lưu mã yêu cầu để cập nhật
      this.currentRequestId = info.requestId
      
      // Hiển thị form
      this.showSupportForm = true
    },

    // Parse thông tin từ tin nhắn thành công
    parseSupportRequestInfo(text) {
      const info = {}
      
      // Parse họ tên
      const fullNameMatch = text.match(/👤 \*\*Họ tên\*\*: (.+)/)
      if (fullNameMatch) info.fullName = fullNameMatch[1]
      
      // Parse tổ chức
      const orgMatch = text.match(/🏢 \*\*Tổ chức\/Công ty\*\*: (.+)/)
      if (orgMatch) info.organization = orgMatch[1]
      
      // Parse số điện thoại
      const phoneMatch = text.match(/📞 \*\*Số điện thoại\*\*: (.+)/)
      if (phoneMatch) info.phone = phoneMatch[1]
      
      // Parse email
      const emailMatch = text.match(/📧 \*\*Email\*\*: (.+)/)
      if (emailMatch) info.email = emailMatch[1]
      
      // Parse mô tả lỗi
      const descMatch = text.match(/🐛 \*\*Mô tả lỗi\*\*: (.+)/)
      if (descMatch) info.issueDescription = descMatch[1]
      
      // Parse mã yêu cầu
      const requestIdMatch = text.match(/🎫 \*\*Mã yêu cầu\*\*: (.+)/)
      if (requestIdMatch) info.requestId = requestIdMatch[1]
      
      return info
    },

    // Cập nhật phương thức handleSupportSubmit để hỗ trợ sửa đổi
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
        
        // Thêm URL ảnh đã upload vào form data
        this.supportForm.images.forEach((image, index) => {
          if (image.url) {
            formData.append(`image_url_${index}`, image.url)
          }
        })

        // Nếu có mã yêu cầu, thêm vào để cập nhật
        if (this.currentRequestId) {
          formData.append('request_id', this.currentRequestId)
        }

        const endpoint = this.currentRequestId 
          ? 'http://localhost:8000/update-support-request'
          : 'http://localhost:8000/support-request'

        const response = await axios.post(
          endpoint,
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
            user_id: user_id,
            requiresSupportForm: false
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
  text-align: left;
  word-wrap: break-word;
  overflow-wrap: break-word;
}

.chat-bubble .bubble p {
  margin: 0.1rem 0;
  text-align: left;
}

.chat-bubble .bubble br {
  margin-bottom: 0.1rem;
  display: block;
  content: "";
}

.chat-bubble .bubble img {
  margin: 0.25rem auto;
  display: block;
  max-width: 100%;
  height: auto;
  border-radius: 8px;
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
  gap: 0.5rem;
}

.input-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.chat-input textarea {
  width: 100%;
  padding: 0.75rem 1rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  resize: none;
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

.upload-btn {
  background: #6c757d !important;
  margin-right: 0.5rem;
}

.upload-btn:hover {
  background: #545b62 !important;
}

.selected-images {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.selected-image-item {
  position: relative;
  display: inline-block;
}

.selected-image-item img {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
  border: 2px solid #42b983;
}

.remove-selected-image {
  position: absolute;
  top: -8px;
  right: -8px;
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

.message-images,
.uploaded-images {
  display: flex;
  gap: 0.5rem;
  margin-top: 8px;
  flex-wrap: wrap;
  justify-content: center;
}

.message-image-item,
.uploaded-image-item {
  position: relative;
  display: inline-block;
  text-align: center;
}

.message-image-item img,
.uploaded-image-item img {
  max-width: 150px;
  max-height: 150px;
  border-radius: 8px;
  object-fit: cover;
  border: 1px solid #ddd;
  display: block;
  margin: 0 auto;
}

.chat-bubble.user .message-image-item img,
.chat-bubble.user .uploaded-image-item img {
  border-color: #42b983;
}

.image-name {
  display: block;
  font-size: 0.8rem;
  color: #666;
  margin-top: 4px;
  text-align: center;
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
  justify-content: center;
}

.image-item {
  position: relative;
  display: inline-block;
  text-align: center;
}

.image-item img {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 8px;
  border: 1px solid #ddd;
  display: block;
  margin: 0 auto;
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

/* Support form button styles */
.support-form-button,
.edit-support-button {
  margin-top: 16px;
  text-align: center;
}

.support-btn,
.edit-support-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 12px 20px;
  border-radius: 25px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
  position: relative;
  overflow: hidden;
}

.edit-support-btn {
  background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
  box-shadow: 0 4px 15px rgba(40, 167, 69, 0.3);
}

.support-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s;
}

.support-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

.edit-support-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(40, 167, 69, 0.4);
}

.support-btn:hover::before,
.edit-support-btn:hover::before {
  left: 100%;
}

.support-btn:active,
.edit-support-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 10px rgba(102, 126, 234, 0.3);
}

.edit-support-btn:active {
  box-shadow: 0 2px 10px rgba(40, 167, 69, 0.3);
}

.btn-icon {
  font-size: 1.1rem;
  filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.1));
}

.btn-text {
  font-weight: 600;
  letter-spacing: 0.5px;
}

.btn-arrow {
  font-size: 1.2rem;
  font-weight: bold;
  transition: transform 0.3s ease;
}

.support-btn:hover .btn-arrow,
.edit-support-btn:hover .btn-arrow {
  transform: translateX(3px);
}

/* Step image styles */
.step-image {
  display: block;
  margin: 4px auto;
  max-width: 100%;
  height: auto;
  border-radius: 8px;
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