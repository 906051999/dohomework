<script setup lang="ts">
import AiAssistant from '../components/AiAssistant.vue'
</script>

<template>
  <div class="home">
    <AiAssistant />
  </div>
</template>

<style scoped>
.home {
  height: 100%;
  width: 100%;
  padding: 0;
  margin: 0;
}

.home :deep(.ai-assistant-fluent) {
  height: 100%;
  width: 100%;
}
</style>