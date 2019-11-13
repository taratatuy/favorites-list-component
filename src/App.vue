<template>
  <div id="app">
    <FavoritesList />
  </div>
</template>

<script>
import FavoritesList from "./components/FavoritesList";

export default {
  name: "App",
  components: {
    FavoritesList
  }
};
</script>

<style>
@import "/bootstrap/bootstrap.css";

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
