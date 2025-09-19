<script setup lang="ts">
import { ref } from 'vue'
import { RouterView } from 'vue-router'

const showAbout = ref(false)
</script>

<template>
  <div id="app">
    <RouterView class="router-view" />
    <div v-if="showAbout" class="about-modal" @click="showAbout = false">
      <div class="about-content" @click.stop>
        <AboutView />
        <button class="close-button" @click="showAbout = false">×</button>
      </div>
    </div>
    <!-- <button class="about-button" @click="showAbout = true">关于</button> -->
  </div>
</template>

<style scoped>
#app {
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  margin: 0;
  padding: 0;
  position: relative;
}

.router-view {
  flex: 1;
  overflow: auto;
}

.about-button {
  position: absolute;
  top: 20px;
  right: 20px;
  padding: 8px 16px;
  background-color: #409eff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  z-index: 100;
}

.about-button:hover {
  background-color: #337ecc;
}

.about-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.about-content {
  position: relative;
  background: white;
  border-radius: 8px;
  max-width: 800px;
  max-height: 80vh;
  overflow-y: auto;
  margin: 20px;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #999;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-button:hover {
  color: #333;
}
</style>