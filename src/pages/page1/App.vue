<template>
  <div id="app">
    <a href="../index/index.html">index</a>
    <h3>page1HTML-page1VUE</h3>
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'index'
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
