<template>
  <div id="app">
    <Scene/>
  </div>
</template>

<script>
import Scene from './components/Scene'

export default {
  name: 'App',
  components: {
    Scene
  }
}
</script>

<style lang="sass">
  
  body
    min-height: 100vh
    margin: 0
    background-image: linear-gradient(to top, #dfe9f3 0%, white 100%)
    overflow: hidden
  
</style>
