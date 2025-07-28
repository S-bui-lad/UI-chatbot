<!-- src/App.vue -->
<template>
  <div id="app">
    <h1>💬 Chatbox AI Chăm Sóc Khách Hàng</h1>
    <Login v-if="!isLoggedIn" @login-success="handleLoginSuccess" />
    <ChatBox v-else />
  </div>
</template>

<script>
import ChatBox from './components/ChatBox.vue'
import Login from './components/Login.vue'

export default {
  components: { ChatBox, Login },
  data() {
    return {
      isLoggedIn: false
    }
  },
  methods: {
    handleLoginSuccess() {
      this.isLoggedIn = true
    }
  }
}
</script>

<style>
#app {
  font-family: 'Segoe UI', sans-serif;
  text-align: center;
  margin-top: 2rem;
}
</style>
