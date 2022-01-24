<template>
  <div id="app" class="app">
    <!-- <img class="logo" alt="Vue logo" src="./assets/logo.png"> -->
    <HelloWorld msg="Tone and Vue.js"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style lang="sass">
  html
    box-sizing: border-box
  *, *:before, *:after
    box-sizing: inherit
  body
    position: relative
    min-height: 100vh
    margin: 0
    padding: 0
  .app
    font-family: Avenir, Helvetica, Arial, sans-serif
    -webkit-font-smoothing: antialiased
    -moz-osx-font-smoothing: grayscale
    text-align: center
    color: #2c3e50
    padding: 60px 0
  .logo
    margin-bottom: 20px
</style>
