<template>
  <div>
    <h3>Графический редактор</h3>
    <CanvasEditor />
  </div>
</template>

<script setup lang="ts">
import CanvasEditor from './components/CanvasEditor.vue';
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 0;
}
</style>