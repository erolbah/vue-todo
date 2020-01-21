<template>
  <div id="app">
    <CheckList/>
  </div>
</template>

<script>
import CheckList from './components/CheckList.vue'

export default {
  name: 'app',
  components: {
    CheckList
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
*{
  box-sizing: border-box;
}
.container {
  max-width: 600px;
  margin: 0 auto;
}
</style>
