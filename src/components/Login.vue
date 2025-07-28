<template>
  <form class="login-form" @submit.prevent="handleLogin">
    <div class="input-group">
      <label for="username">Tên đăng nhập</label>
      <input
        type="text"
        v-model="username"
        id="username"
        placeholder="Tên đăng nhập"
        required
        autocomplete="username"
      />
    </div>
    <div class="input-group">
      <label for="password">Mật khẩu</label>
      <input
        type="password"
        v-model="password"
        id="password"
        placeholder="Mật khẩu"
        required
        autocomplete="current-password"
      />
    </div>
    <button class="login-btn" type="submit">Đăng nhập</button>
    <div v-if="error" class="error">{{ error }}</div>
  </form>
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

    const handleLogin = async () => {
      error.value = '';
      try {
        await login(username.value, password.value);
        emit('login-success'); // Đúng cú pháp emit trong setup
      } catch (e) {
        error.value = 'Sai tài khoản hoặc mật khẩu';
      }
    };

    return {
      username,
      password,
      error,
      handleLogin,
    };
  },
};
</script>

<style scoped>
.login-form {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 1.2rem;
  background: transparent;
  box-shadow: none;
  border: none;
  padding: 0;
}
.input-group {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}
.input-group label {
  font-size: 0.98rem;
  color: #444;
  font-weight: 400;
  margin-bottom: 0.1rem;
}
.input-group input {
  border: none;
  border-bottom: 1.5px solid #e5e7eb;
  background: #fff;
  padding: 0.7rem 0.2rem 0.5rem 0.2rem;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.2s;
  border-radius: 0;
}
.input-group input:focus {
  border-bottom: 1.5px solid #60a5fa;
  background: #f9fafb;
}
.login-btn {
  width: 100%;
  padding: 0.8rem 0;
  background: #e0e7ef;
  color: #222;
  font-size: 1.05rem;
  font-weight: 500;
  border: none;
  border-radius: 7px;
  cursor: pointer;
  margin-top: 0.2rem;
  transition: background 0.2s;
}
.login-btn:hover {
  background: #bcd2f6;
}
.error {
  color: #ef4444;
  background: #fee2e2;
  border-radius: 6px;
  padding: 0.5rem 1rem;
  margin-top: 0.5rem;
  font-size: 0.98rem;
  text-align: center;
}
</style>