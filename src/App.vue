<template>
  <div id="app">
    <!--<Hero/>-->
    <Slots/>
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
      showApp: true,
    }
  },
  mounted(){
    document.body.addEventListener("click", this.stopCoins);

    this.checkScreen();
    // window.addEventListener("resize", this.checkScreen)
  },
  methods:{
    checkScreen(){
      // this.showApp = window.innerWidth > 900;
    },
    stopCoins(){
      const coinrain = document.getElementById('coinrain');
      if(coinrain){
        document.body.removeChild(coinrain)
      }
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
  }

  body {
    min-height: 100vh;
    display: flex;
    flex-direction: row-reverse;
    justify-content: center;
    align-items: center;
    background: #272727;
    margin: 0;
  }

  canvas {
    z-index: 0;
  }

  canvas#coinrain {
    display: block;
    pointer-events:none;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 12;
    width: 100%;
  }
</style>
