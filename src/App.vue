<template>
  <div class="app">
    <flow-canvas />
  </div>
</template>

<script>
import FlowCanvas from './components/FlowCanvas.vue'

export default {
  name: 'App',
  components: {
    FlowCanvas
  }
}
</script>

<style>
:root {
  --background-dark: #1E1E1E;
  --text-color: #FFFFFF;
}

body {
  margin: 0;
  padding: 0;
  background-color: var(--background-dark);
  color: var(--text-color);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell;
  overflow: hidden;
}

.app {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
</style>
