<script setup>
import Snake from './components/Snake.vue'
</script>

<template>
  <div class="app-container">
    <h1>Vue 贪吃蛇游戏</h1>
    <Snake />
  </div>
</template>

<style scoped>
.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

h1 {
  color: #2c3e50;
  margin-bottom: 30px;
}
</style>
