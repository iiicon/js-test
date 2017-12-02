<template>
  <div id="app">
    <img src="./assets/logo.png">
    <vue-area :props-show="show" :props-result="result" v-on:result="areaResult"></vue-area>
    <router-view></router-view>
  </div>
</template>

<script>
import vueArea from 'vue-area'
export default {
  name: 'app',
  data() {
    return {
      result: null,
      show: false
    }
  },
  components: {
    vueArea
  },
  methods: {
    areaResult: function(show, result) {
      this.show = show
      this.result = result
    }
  },
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
