<template>
  <div id="app">

    <!-- <div class="item" ref="item"></div>
    <span>Scroll down to rotate</span>
    <div class="circle" ref="circle"></div>
    <div class="circleBlack" ref="circleBlack"></div> -->

    <!-- <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div> -->
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  font-size: 16px;
  overflow-x: hidden;
  background: black;
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

canvas {
  position: fixed;
  pointer-events: none;
  top: 0;
}

@font-face {
  font-family: 'shar-medium';
  src: url('./assets/fonts/ss-500.woff') format('woff');
}
</style>
