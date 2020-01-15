<template>
  <div id="app">
    <img src="./assets/logo.png">
    <AgentBar />
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import AgentBar from './components/AgentBar'

export default {
  name: 'app',
  components: {
    // HelloWorld
    AgentBar
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
</style>
