<template>
  <div id="app">
    <Simbase/>
  </div>
</template>

<script>
import Simbase from './components/Simbase.vue'

export default {
  name: 'App',
  components: {
    Simbase
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
