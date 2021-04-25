<template>
  <div id="app">
    <Slots />
  </div>
</template>

<script>
import Slots from './components/Slots.vue'

export default {
  name: 'App',
  components: {
    Slots
  }
}
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
