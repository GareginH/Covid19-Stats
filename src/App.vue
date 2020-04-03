<template>
  <div id="app">
    <covidScrapper />
  </div>
</template>

<script>
import covidScrapper from "./components/covidScrapper";
export default {
  name: "App",
  components: {
    covidScrapper
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
body {
  margin: 0px 0px 20px 0px;

  background-color: transparent;
}
</style>
