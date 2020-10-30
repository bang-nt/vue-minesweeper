<template>
  <div id="app">
    <Game />
    <Foot />
  </div>
</template>

<script>
import Game from "./components/Game";
import Foot from "./components/Foot";
export default {
  name: "App",
  components: {
    Game,
    Foot,
  },
};
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Roboto&family=Ubuntu:wght@700&display=swap");

$background-color: #006989;
$tile-color: #d4d4d4;
$border-radius: calc(var(--size) / 10);

html {
  height: 100%;
}

body {
  // background-color: $background-color;
  font-family: "Roboto", sans-serif;
  // height: 100%;
  margin: 0;
  text-align: center;
}
#app {
  margin: 0;
  // width: auto;
  height: 100%;
  font-family: "Lemonada", cursive;
  // background: url(bg-home.jpg);
  background-image: url("~@/assets/app-background.jpg");
  display: flex;
  flex-direction: column;
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
  align-items: center;
  justify-content: center;
}
</style>
