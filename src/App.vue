<template>
  <div id="app">
    <h1>vue-draw</h1>
    <div class="show-area">
      <CanvasDraw :width="640" :height="480" :outputName="'example'"/>
    </div>
  </div>
</template>

<script>
import CanvasDraw from './components/CanvasDraw.vue'

export default {
  name: 'app',
  components: {
    CanvasDraw
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.show-area {
  width: fit-content;
  margin: 0 auto;
}
</style>
