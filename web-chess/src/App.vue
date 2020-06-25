<template>
  <div id="app">
    <HelloWorld msg=""/>
    <!-- <test></test> -->
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Test from './components/test.vue'

export default {
  name: 'app',
  components: {
    HelloWorld,
    Test
  }
}
</script>

<style>
</style>
