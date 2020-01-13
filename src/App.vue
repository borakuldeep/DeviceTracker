<template>
  <div id="app">
    <Map></Map>
  </div>
</template>

<script>
import Map from "./components/Map.vue";
//import Notification from "./components/NotificationPanel.vue";

export default {
  name: "app",
  components: {
    Map
    //Notification
  }
};
</script>

<style>
#app2 {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

body {
  margin: 0;
  padding: 0;
  font-family: "Roboto", sans-serif;
}
</style>
