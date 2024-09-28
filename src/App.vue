<template>
  <div id="app">
    <ElavatorSimulator msg="Welcome to nhom nam 24.1"/>
  </div>
</template>

<script>
import ElavatorSimulator from './components/ElavatorSimulator.vue';

export default {
  name: 'App',
  components: {
    ElavatorSimulator
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
