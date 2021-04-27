<template>
  <div class="header">
    <h1>2048</h1>
    <p>
      Get the <strong>2048</strong> tile to win
    </p>
    <p>
      Use arrow keys or swipe on touch screen to move tiles
    </p>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'Header',
})
</script>

<style lang="scss" scoped>
@import '../styles/variables.scss';

.header {
  width: $container-size;
}
</style>
