<template>
  <div class="game-container">
    <Header />
    <Board />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Board from './components/Board.vue'
import Header from './components/Header.vue'

export default defineComponent({
  name: 'App',
  components: {
    Board,
    Header,
  }
})
</script>

<style scoped>
.game-container {
  font-family: Verdana;
  display: flex;
  align-items: center;
  flex-direction: column;
}
</style>
