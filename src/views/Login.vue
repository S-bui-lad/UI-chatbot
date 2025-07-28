<template>
  <div class="login-bg">
    <div class="login-card">
      <div class="logo-wrap">
        <img src="@/assets/logo.png" alt="Logo" class="logo" />
      </div>
      <div class="title">Đăng nhập Chatbox AI</div>
      <div class="desc">Đăng nhập để bắt đầu trò chuyện với AI</div>
      <LoginForm @login="handleLogin" :loginError="loginError" />
    </div>
  </div>
</template>

<script>
import LoginForm from '../components/Login.vue';
import { login } from '../services/auth';

export default {
  components: {
    LoginForm
  },
  data() {
    return {
      loginError: ''
    }
  },
  methods: {
    async handleLogin(credentials) {
      this.loginError = '';
      try {
        await login(credentials.username, credentials.password);
        this.$router.push({ name: 'Dashboard' });
      } catch (error) {
        this.loginError = 'Tên đăng nhập hoặc mật khẩu không đúng.';
      }
    }
  }
};
</script>

<style scoped>
.login-bg {
  min-height: 100vh;
  background: #ececf1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Inter', 'Segoe UI', Arial, sans-serif;
}
.login-card {
  background: #fff;
  border-radius: 28px;
  box-shadow: 0 2px 12px 0 rgba(60,60,120,0.06);
  padding: 2.5rem 2.2rem 2rem 2.2rem;
  min-width: 320px;
  max-width: 350px;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.2rem;
  border: none;
}
.logo-wrap {
  background: #f4f6fa;
  border-radius: 50%;
  padding: 0.8rem;
  margin-bottom: 0.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
}
.logo {
  width: 44px;
  height: 44px;
  object-fit: contain;
  display: block;
}
.title {
  font-size: 1.32rem;
  font-weight: 700;
  color: #23272f;
  margin-bottom: 0.1rem;
  letter-spacing: 0.01em;
  text-align: center;
}
.desc {
  color: #7b7e87;
  margin-bottom: 0.2rem;
  font-size: 1rem;
  font-weight: 400;
  text-align: center;
}
</style>