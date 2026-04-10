<script setup lang="ts">
// import HelloWorld from './components/HelloWorld.vue'
// import TravelApp from './components/TravelApp.vue'
</script>

<template>
  <!-- <div>
    <a href="https://vite.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div> -->
  <!-- <HelloWorld msg="Vite + Vue" /> -->
  <!-- <TravelApp msg="Travel Tracker" /> -->
  
  <div id="app">
    <!-- 导航区域 -->
     <router-view/>
  </div>
</template>

<style>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  /* height: 100%; */
  /* 关键：禁止横向滚动，防止手机端出现左右拖动 */
  overflow-x: hidden; 
  /* 优化移动端点击高亮颜色 */
  -webkit-tap-highlight-color: transparent;
}

#app {
  width: 100%;
  min-height: 100vh; /* 确保至少占满一屏高度 */
  display: flex;
  flex-direction: column;
  /* 字体优化，提升移动端阅读体验 */
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
