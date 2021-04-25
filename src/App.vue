<template>
  <div id="app">
    <Hero
      v-if="!showApp"
    />
    <Slots
      v-if="showApp"
    />
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
  },
  data(){
    return {
      showApp: null,
    }
  },
  mounted(){
    this.checkScreen();
    window.addEventListener("resize", this.checkScreen)
  },
  methods:{
    checkScreen(){
      this.showApp = window.innerWidth > 900;
    }
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
    flex-direction: row-reverse;
    justify-content: center;
    align-items: center;
  }
</style>
