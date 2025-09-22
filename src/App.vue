<template>
  <div id="app">
    <h1>Графический редактор</h1>
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
  margin-top: 60px;
}
</style>