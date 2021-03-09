<template>
  <div id="app">
    <Translator />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Translator from "./components/Translator.vue";

@Component({
  components: {
    Translator
  }
})
export default class App extends Vue {}
</script>

<style scoped>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
