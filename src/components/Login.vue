<template>
  <div class="login-container">
    <div class="login-card">
      <div class="login-header">
        <div class="logo">
          <img src="../assets/logo.png" alt="Logo" class="logo-img" />
        </div>
        <h1 class="welcome-text">Chào mừng bạn trở lại!</h1>
        <p class="subtitle">Đăng nhập để tiếp tục sử dụng dịch vụ</p>
      </div>

      <form class="login-form" @submit.prevent="handleLogin">
        <div class="input-group">
          <label for="username" class="input-label">
            <i class="icon user-icon">👤</i>
            Tên đăng nhập
          </label>
          <input
            type="text"
            v-model="username"
            id="username"
            placeholder="Nhập tên đăng nhập của bạn"
            required
            autocomplete="username"
            :class="{ 'input-error': usernameError }"
            @focus="clearUsernameError"
            @blur="validateUsername"
          />
          <span v-if="usernameError" class="error-message">{{ usernameError }}</span>
        </div>

        <div class="input-group">
          <label for="password" class="input-label">
            <i class="icon lock-icon">🔒</i>
            Mật khẩu
          </label>
          <div class="password-input-wrapper">
            <input
              :type="showPassword ? 'text' : 'password'"
              v-model="password"
              id="password"
              placeholder="Nhập mật khẩu của bạn"
              required
              autocomplete="current-password"
              :class="{ 'input-error': passwordError }"
              @focus="clearPasswordError"
              @blur="validatePassword"
            />
            <button
              type="button"
              class="password-toggle"
              @click="togglePassword"
              :title="showPassword ? 'Ẩn mật khẩu' : 'Hiện mật khẩu'"
            >
              {{ showPassword ? '🙈' : '👁️' }}
            </button>
          </div>
          <span v-if="passwordError" class="error-message">{{ passwordError }}</span>
        </div>

        <div class="form-options">
          <label class="remember-me">
            <input type="checkbox" v-model="rememberMe" />
            <span class="checkmark"></span>
            Ghi nhớ đăng nhập
          </label>
          <a href="#" class="forgot-password">Quên mật khẩu?</a>
        </div>

        <button 
          class="login-btn" 
          type="submit" 
          :disabled="isLoading"
          :class="{ 'loading': isLoading }"
        >
          <span v-if="!isLoading">Đăng nhập</span>
          <span v-else class="loading-text">
            <span class="spinner"></span>
            Đang đăng nhập...
          </span>
        </button>

        <div v-if="error" class="error-alert">
          <div class="error-icon">⚠️</div>
          <div class="error-content">
            <h4>Đăng nhập thất bại</h4>
            <p>{{ error }}</p>
          </div>
        </div>
      </form>

      <div class="login-footer">
        <p>Chưa có tài khoản? <a href="#" class="signup-link">Đăng ký ngay</a></p>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import { login } from '../services/auth';

export default {
  name: 'LoginForm',
  setup(props, { emit }) {
    const username = ref('');
    const password = ref('');
    const error = ref('');
    const isLoading = ref(false);
    const showPassword = ref(false);
    const rememberMe = ref(false);
    const usernameError = ref('');
    const passwordError = ref('');

    const validateUsername = () => {
      if (!username.value.trim()) {
        usernameError.value = 'Vui lòng nhập tên đăng nhập';
        return false;
      }
      if (username.value.length < 3) {
        usernameError.value = 'Tên đăng nhập phải có ít nhất 3 ký tự';
        return false;
      }
      usernameError.value = '';
      return true;
    };

    const validatePassword = () => {
      if (!password.value) {
        passwordError.value = 'Vui lòng nhập mật khẩu';
        return false;
      }
      if (password.value.length < 6) {
        passwordError.value = 'Mật khẩu phải có ít nhất 6 ký tự';
        return false;
      }
      passwordError.value = '';
      return true;
    };

    const clearUsernameError = () => {
      usernameError.value = '';
    };

    const clearPasswordError = () => {
      passwordError.value = '';
    };

    const togglePassword = () => {
      showPassword.value = !showPassword.value;
    };

    const handleLogin = async () => {
      // Clear previous errors
      error.value = '';
      usernameError.value = '';
      passwordError.value = '';

      // Validate inputs
      const isUsernameValid = validateUsername();
      const isPasswordValid = validatePassword();

      if (!isUsernameValid || !isPasswordValid) {
        return;
      }

      isLoading.value = true;

      try {
        await login(username.value, password.value);
        emit('login-success');
      } catch (e) {
        error.value = 'Tên đăng nhập hoặc mật khẩu không chính xác. Vui lòng thử lại.';
      } finally {
        isLoading.value = false;
      }
    };

    return {
      username,
      password,
      error,
      isLoading,
      showPassword,
      rememberMe,
      usernameError,
      passwordError,
      handleLogin,
      togglePassword,
      validateUsername,
      validatePassword,
      clearUsernameError,
      clearPasswordError,
    };
  },
};
</script>

<style scoped>
.login-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 1rem;
}

.login-card {
  background: white;
  border-radius: 20px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  padding: 2.5rem;
  width: 100%;
  max-width: 400px;
  animation: slideUp 0.6s ease-out;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.login-header {
  text-align: center;
  margin-bottom: 2rem;
}

.logo {
  margin-bottom: 1rem;
}

.logo-img {
  width: 60px;
  height: 60px;
  object-fit: contain;
}

.welcome-text {
  font-size: 1.8rem;
  font-weight: 700;
  color: #1f2937;
  margin: 0 0 0.5rem 0;
}

.subtitle {
  color: #6b7280;
  font-size: 1rem;
  margin: 0;
}

.login-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.input-label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.95rem;
  font-weight: 500;
  color: #374151;
}

.icon {
  font-size: 1.1rem;
}

.input-group input {
  border: 2px solid #e5e7eb;
  background: #f9fafb;
  padding: 0.875rem 1rem;
  font-size: 1rem;
  border-radius: 12px;
  outline: none;
  transition: all 0.3s ease;
  color: #1f2937;
}

.input-group input:focus {
  border-color: #3b82f6;
  background: white;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.input-group input.input-error {
  border-color: #ef4444;
  background: #fef2f2;
}

.input-group input.input-error:focus {
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.1);
}

.password-input-wrapper {
  position: relative;
  width: 100%;
}

.password-input-wrapper input {
  width: 100%;
  padding-right: 3rem;
  box-sizing: border-box;
}

.password-toggle {
  position: absolute;
  right: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  font-size: 1.1rem;
  cursor: pointer;
  padding: 0.25rem;
  border-radius: 4px;
  transition: background-color 0.2s;
  z-index: 10;
}

.password-toggle:hover {
  background: rgba(0, 0, 0, 0.05);
}

.error-message {
  color: #ef4444;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}

.form-options {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 0.5rem 0;
}

.remember-me {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.9rem;
  color: #6b7280;
  cursor: pointer;
}

.remember-me input[type="checkbox"] {
  display: none;
}

.checkmark {
  width: 18px;
  height: 18px;
  border: 2px solid #d1d5db;
  border-radius: 4px;
  position: relative;
  transition: all 0.2s;
}

.remember-me input[type="checkbox"]:checked + .checkmark {
  background: #3b82f6;
  border-color: #3b82f6;
}

.remember-me input[type="checkbox"]:checked + .checkmark::after {
  content: '✓';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 0.75rem;
  font-weight: bold;
}

.forgot-password {
  color: #3b82f6;
  text-decoration: none;
  font-size: 0.9rem;
  transition: color 0.2s;
}

.forgot-password:hover {
  color: #2563eb;
  text-decoration: underline;
}

.login-btn {
  width: 100%;
  padding: 1rem;
  background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
  color: white;
  font-size: 1.1rem;
  font-weight: 600;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.login-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(59, 130, 246, 0.3);
}

.login-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

.login-btn.loading {
  background: linear-gradient(135deg, #6b7280 0%, #4b5563 100%);
}

.loading-text {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top: 2px solid white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-alert {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  background: #fef2f2;
  border: 1px solid #fecaca;
  border-radius: 12px;
  padding: 1rem;
  margin-top: 1rem;
  animation: shake 0.5s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

.error-icon {
  font-size: 1.2rem;
  flex-shrink: 0;
}

.error-content h4 {
  margin: 0 0 0.25rem 0;
  color: #dc2626;
  font-size: 0.95rem;
  font-weight: 600;
}

.error-content p {
  margin: 0;
  color: #7f1d1d;
  font-size: 0.9rem;
  line-height: 1.4;
}

.login-footer {
  text-align: center;
  margin-top: 2rem;
  padding-top: 1.5rem;
  border-top: 1px solid #e5e7eb;
}

.login-footer p {
  color: #6b7280;
  font-size: 0.9rem;
  margin: 0;
}

.signup-link {
  color: #3b82f6;
  text-decoration: none;
  font-weight: 500;
  transition: color 0.2s;
}

.signup-link:hover {
  color: #2563eb;
  text-decoration: underline;
}

/* Responsive design */
@media (max-width: 480px) {
  .login-card {
    padding: 2rem 1.5rem;
    margin: 1rem;
  }
  
  .welcome-text {
    font-size: 1.5rem;
  }
  
  .form-options {
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }
}
</style>