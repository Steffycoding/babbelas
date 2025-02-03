<template>
  <div id="app">
    <GameBoard :players="players" />
  </div>
</template>

<script>
import GameBoard from './components/GameBoard.vue';

export default {
  components: {
    GameBoard
  },
  data() {
    return {
      players: [] // Players will be passed down
    };
  }
};
</script>

<style>
#app {
  text-align: center;
}
</style>
