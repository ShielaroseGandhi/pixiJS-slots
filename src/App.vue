<template>
  <div id="app">
    <Hero />
    <Slots />
  </div>
</template>

<script>
import Slots from './components/Slots.vue'
import Hero from "./components/Hero";

export default {
  name: 'App',
  components: {
    Hero,
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
  body {
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
</style>
