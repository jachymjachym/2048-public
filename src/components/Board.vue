<script lang="ts">
import { defineComponent, ref, onMounted, onUnmounted } from 'vue'

enum Color {
  tile2 = '#FFC43D',
  tile4 = '#EF476F',
  tile8 = '#1B9AAA',
  tile16 = '#06D6A0',
  tile32 = 'blue',
  tile64 = 'green',
  tile128 = 'pink',
  tile256 = 'red',
  tile512 = 'violet',
  tile1024 = 'brown',
  tile2048 = 'gold',
}

enum Direction {
  RIGHT = 'right',
  LEFT = 'left',
  UP = 'up',
  DOWN = 'down',
}

interface Tile {
  x: number,
  y: number,
  value: number,
}

export default defineComponent({
  name: 'Board',
  props: {
    
  },
  setup: () => {
    const size = 4;
    const tiles = ref<Tile[]>([
      {x: 1, y: 3, value: 2},
      {x: 1, y: 2, value: 4},
      {x: 1, y: 1, value: 8},
      {x: 1, y: 4, value: 16},
      {x: 2, y: 1, value: 32},
      {x: 3, y: 1, value: 64},
    ]);

    const sortTiles = (direction: Direction, tiles: Tile[]) => {
      switch(direction) {
        case Direction.RIGHT: 
          return tiles.sort((a, b) => {return b.x-a.x}) // Descendant sort
          break;
        // case Direction.DOWN: 
        //   return tiles.sort((a, b) => {return a.y-b.y}) // Ascendant sort
        //   break;
        case Direction.LEFT: 
          return tiles.sort((a, b) => {return a.x-b.x}) // Ascendant sort
          break;
        // case Direction.UP: 
        //   return tiles.sort((a, b) => {return b.y-a.y}) // Descendant sort
        //   break;
        default: 
          return [];
      }
    };

    const moveRight = (direction: Direction): void => {
      const axis = (direction === Direction.RIGHT || direction === Direction.LEFT) ? 'x' : 'y';
      const mood = (direction === Direction.RIGHT || direction === Direction.UP) ? 1 : -1; // positive or negative mood
      
      for(let i: any = 1; i <= size; i++) {
        // Filter tiles for each row and sort
        const tilesInRow: Tile[] = tiles.value.filter((tile) => {
          if(direction === Direction.RIGHT || direction === Direction.LEFT) {
            return i === tile.y;
          } else {
            return i === tile.x;
          }
        });

        let moveLenght: number = 4 - tilesInRow.length;

        sortTiles(direction, tilesInRow).map((tile, index, sortedTiles) => {
          const isNextTileMergeable: boolean = !!sortedTiles[index - 1] && sortedTiles[index - 1].value === tile.value;
          const isNextTileOccupied: boolean = !!sortedTiles[index - 1] && sortedTiles[index - 1][axis] === tile[axis] + 1;
          
          if(tile[axis] === 4 || (isNextTileOccupied && !isNextTileMergeable)) {
            return
          } else if(isNextTileMergeable) {
            // Merge tiles
            sortedTiles[index - 1].value = sortedTiles[index - 1].value * 2;
            tiles.value.splice(tiles.value.findIndex((target) => target[axis] === tile[axis]), 1);
            moveLenght += 1;
          }

          tile[axis] += moveLenght * mood;
        });
      }
    };

    const checkKey = (event: any) => {
      const keydownEvent = event || window.event;

      if (event.keyCode == '38') {
        moveRight(Direction.UP);
      }
      else if (event.keyCode == '40') {
        moveRight(Direction.DOWN);
      }
      else if (event.keyCode == '37') {
        moveRight(Direction.LEFT);
      }
      else if (event.keyCode == '39') {
        moveRight(Direction.RIGHT);
      }

    }

    onMounted(() => {
      document.onkeydown = checkKey;
    });

    onUnmounted(() => {
      document.onkeydown = null;
    })

    return {
      size,
      tiles,
      Color,
    };
  }
})
</script>

<template>
  <div class="wrapper">
    <div class="container">
      <div v-for="(n, index) in size*size" :key="index" class="block"></div>
    </div>
    <div class="tile-container">
      <div 
        v-for="(tile, index) in tiles"
        :key="index"
        class="tile"
        :style="{ 
          top: (tile.y - 1)*100 + 'px',
          left: (tile.x - 1)*100 + 'px',
          backgroundColor: Color[`tile${tile.value}`]
        }">
        {{ tile.value }}
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
$size: 100px;

.wrapper {
  position: relative;
}

.container {
  display: flex;
  flex-wrap: wrap;
  width: $size*4;
  height: $size*4;
}

.block {
  width: $size;
  height: $size;
  background: #999999;
  box-shadow: 1px 1px inset black, -1px -1px inset black;
}

.tile-container {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
}

.tile {
  position: absolute;
  width: $size;
  height: $size;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 32px;
  color: white;
}
</style>
