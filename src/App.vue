<template>
  <div id="app">
    <Dashboard/>
  </div>
</template>

<script>
import Dashboard from './components/Dashboard.vue'

export default {
  name: 'app',
  components: {
    Dashboard
  }
}
</script>

<style lang="scss">
@font-face {
  font-family: "OstrichSans";
  src: url("assets/fonts/OstrichSans-Medium.otf");
}

@font-face {
  font-family: "OstrichSansBlack";
  src: url("assets/fonts/OstrichSans-Black.otf");
}

#app {
  font-family: 'OstrichSansBlack', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
