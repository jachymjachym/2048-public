<script lang="ts">
import { defineComponent, ref, computed, onMounted, onUnmounted, ComputedRef } from 'vue';
import SwipeDetect from './SwipeDetect.vue'

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
  id: string,
}

export default defineComponent({
  name: 'Board',
  components: {
    SwipeDetect,
  },
  setup: () => {
    const gridItem = ref(null);
    const tileWidth = ref(0);
    const id = ref(1);
    const size = 4;
    const winningNumber = 2048;
    const tiles = ref<Tile[]>([]);
    const defeat = ref(false);
    const gameWon = ref(false);

    const initTile = () => {
      tiles.value = [];
      tiles.value.push({
        x: Math.floor(Math.random() * size + 1),
        y: Math.floor(Math.random() * size + 1),
        value: 2,
        id: `tile${id.value}`
      });
    };

    const findEmptyCells = () => {
      const cells = [];
      for(let x: any = 1; x <= size; x++) {
        for(let y: any = 1; y <= size; y++) {
          if(!tiles.value.find((tile) => tile.x === x && tile.y === y)) {
            cells.push({x, y});
          }
        }
      }
      return cells;
    };

    const isPossibleMove = (): boolean => {
      const isFullBoard = findEmptyCells().length === 0;
      let isPossible = false;

      tiles.value.forEach((item, index, tiles) => {
        const currentTileX = item.x;
        const currentTileY = item.y;
        const currentTileValue = item.value;

        isPossible = (
          !isFullBoard ||
          isFullBoard &&
          (currentTileValue === tiles.find((tile) => tile.x === currentTileX + 1 && tile.y === currentTileY)?.value ||
          currentTileValue === tiles.find((tile) => tile.x === currentTileX - 1 && tile.y === currentTileY)?.value ||
          currentTileValue === tiles.find((tile) => tile.y === currentTileY + 1 && tile.x === currentTileX)?.value ||
          currentTileValue === tiles.find((tile) => tile.y === currentTileY - 1 && tile.x === currentTileX)?.value)
        );
      });

      return isPossible;
    };

    const generateNewTile = () => {
      const tileValue = Math.random() > 0.9 ? 4 : 2;
      const randomEmptyCell = findEmptyCells()[Math.floor(Math.random() * findEmptyCells().length)];
      id.value += 1;
      tiles.value.push({
        x: randomEmptyCell.x,
        y: randomEmptyCell.y,
        value: tileValue,
        id: `tile${id.value}`
      });

      if(!isPossibleMove()) {
        defeat.value = true;
      }
    };

    const resetGame = () => {
      defeat.value = false;
      gameWon.value = false;
      initTile();
    };

    const sortCallback = (direction: Direction) => {
      switch(direction) {
        case Direction.RIGHT: 
          return (a: Tile, b: Tile) => {return b.x-a.x} // Descendant sort
          break;
        case Direction.DOWN: 
          return (a: Tile, b: Tile) => {return b.y-a.y} // Descendant sort
          break;
        case Direction.LEFT: 
          return (a: Tile, b: Tile) => {return a.x-b.x} // Ascendant sort
          break;
        case Direction.UP: 
          return (a: Tile, b: Tile) => {return a.y-b.y} // Ascendant sort
          break;
      }
    };

    const indexOfTile = (id: string) => tiles.value.findIndex((target) => id === target.id);

    const moveTiles = (direction: Direction): void => {
      const axis = (direction === Direction.RIGHT || direction === Direction.LEFT) ? 'x' : 'y';
      const mood = (direction === Direction.RIGHT || direction === Direction.DOWN) ? 1 : -1; // positive or negative mood
      const limitPosition = (direction === Direction.RIGHT || direction === Direction.DOWN) ? 4 : 1;
      let gotUpdated = false;
      
      for(let i: any = 1; i <= size; i++) {
        // Filter tiles for each row and sort
        const tilesInRow: ComputedRef<Tile[]> = computed(() => tiles.value.filter((tile) => {
          if(direction === Direction.RIGHT || direction === Direction.LEFT) {
            return i === tile.y;
          } else {
            return i === tile.x;
          }
        }).sort(sortCallback(direction)));

        tilesInRow.value.map((tile, index, sortedTiles) => {
          const isNextTileMergeable = !!sortedTiles[index - 1] && sortedTiles[index - 1].value === tile.value;
          const isNextNextTileMergeable = !!sortedTiles[index - 2] && sortedTiles[index - 2].value === tile.value;
          const isNextPositionOccupied = !!sortedTiles[index - 1] && sortedTiles[index - 1][axis] === tile[axis] + mood;
          
          if(isNextTileMergeable && !!tiles.value[indexOfTile(sortedTiles[index - 1].id)]) {
            tile[axis] = !!sortedTiles[index - 1] ? sortedTiles[index - 1][axis]: limitPosition;
            
            tiles.value[indexOfTile(sortedTiles[index - 1].id)].value *= 2;
            tiles.value[indexOfTile(tile.id)].value *= 2;

            if(tiles.value[indexOfTile(tile.id)].value === winningNumber) {
              gameWon.value = true;
            }

            tiles.value.splice(indexOfTile(tile.id), 1);
            gotUpdated = true;
          } else if(
            (tile[axis] !== limitPosition && !isNextPositionOccupied) || 
            (isNextTileMergeable && isNextNextTileMergeable)
          ) {
            tile[axis] = !!sortedTiles[index - 1] ? sortedTiles[index - 1][axis] - mood : limitPosition;
            gotUpdated = true;
          }
        });
      }

      if(gotUpdated) {
        generateNewTile();
      }
      
    };

    const checkKey = (event: any) => {
      const keydownEvent = event || window.event;

      if (event.keyCode == '38') {
        moveTiles(Direction.UP);
      }
      else if (event.keyCode == '40') {
        moveTiles(Direction.DOWN);
      }
      else if (event.keyCode == '37') {
        moveTiles(Direction.LEFT);
      }
      else if (event.keyCode == '39') {
        moveTiles(Direction.RIGHT);
      }

    }

    onMounted(() => {
      document.onkeydown = checkKey;
      tileWidth.value = gridItem.value.offsetWidth;
      initTile();
    });

    onUnmounted(() => {
      document.onkeydown = null;
    })

    return {
      size,
      tiles,
      defeat,
      gameWon,
      resetGame,
      moveTiles,
      gridItem,
      tileWidth,
    };
  }
})
</script>

<template>
  <div class="wrapper">
    <div class="container">
      <div ref="gridItem" v-for="(n, index) in size*size" :key="index" class="grid-item"></div>
    </div>
    <SwipeDetect :callback="moveTiles">
      <div class="tile-container">
        <div 
          v-for="(tile, index) in tiles"
          :key="index"
          class="tile"
          :style="{ 
            top: (tile.y - 1) * (tileWidth + 10) + 5 + 'px',
            left: (tile.x - 1) * (tileWidth + 10) + 5 + 'px',
          }"
          :class="`tile-${tile.value}`">
          {{ tile.value }}
        </div>
      </div>
    </SwipeDetect>
    <div v-if="defeat" class="game-over"> 
      <div class="announcement">
        You lose!
      </div>
      <button class="btn" @click="resetGame">Try again</button>
    </div>
    <div v-if="gameWon" class="game-over"> 
      <div class="announcement">
        You win!
      </div>
      <button class="btn" @click="resetGame">Do it again</button>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import '../styles/variables.scss';

.tile-2 { background: $color-2; }
.tile-4 { background: $color-4; }
.tile-8 { background: $color-8; }
.tile-16 { background: $color-16; }
.tile-32 { background: $color-32; }
.tile-64 { background: $color-64; }
.tile-128 { background: $color-128; }
.tile-256 { background: $color-256; }
.tile-512 { background: $color-512; }
.tile-1024 { background: $color-1024; }
.tile-2048 { background: $color-2048; }

.wrapper {
  position: relative;
}

.container {
  display: flex;
  flex-wrap: wrap;
  width: $container-size-sm;
  height: $container-size-sm;
  background: $color-container;
  padding: $spacing;

  @media only screen and (min-width: 500px) {
    width: $container-size-lg;
    height: $container-size-lg;
  }
}

.grid-item {
  width: $size-sm;
  height: $size-sm;
  margin: $spacing;
  background: $color-background;

  @media only screen and (min-width: 500px) {
    width: $size-lg;
    height: $size-lg;
  }
}

.tile-container {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
}

.game-over {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: $color-game-over-background;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.announcement {
  font-size: 32px;
  margin-bottom: 16px;
}

.btn {
  background: $color-black;
  display: inline-block;
  border: none;
  padding: 1rem 2rem;
  margin: 0;
  text-decoration: none;
  color: $color-white;
  font-family: sans-serif;
  cursor: pointer;
  text-align: center;
}

.tile {
  position: absolute;
  width: $size-sm;
  height: $size-sm;
  margin: $spacing;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 32px;
  color: $color-white;

  @media only screen and (min-width: 500px) {
    width: $size-lg;
    height: $size-lg;
  }
}
</style>
