<!-- src/App.vue -->
<template>
  <div id="app">
    <h1>💬 Chatbox AI Chăm Sóc Khách Hàng</h1>
    <ChatBox />
  </div>
</template>

<script>
import ChatBox from './components/ChatBox.vue'

export default {
  components: { ChatBox }
}
</script>

<style>
#app {
  font-family: 'Segoe UI', sans-serif;
  text-align: center;
  margin-top: 2rem;
}
</style>
