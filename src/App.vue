<template>
  <div class="game-container">
    <Board />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Board from './components/Board.vue'

export default defineComponent({
  name: 'App',
  components: {
    Board,
  }
})
</script>

<style scoped>
.game-container {
  display: flex;
  justify-content: center;
}
</style>
