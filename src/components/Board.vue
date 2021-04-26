<script lang="ts">
import { defineComponent, ref, computed, onMounted, onUnmounted, ComputedRef } from 'vue'

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
  id: string,
}

export default defineComponent({
  name: 'Board',
  props: {
    
  },
  setup: () => {
    const id = ref(1);
    const size = 4;
    const tiles = ref<Tile[]>([
      {x: 2, y: 1, value: 2, id: `mrdka${id.value}` },
    ]);

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

    const generateNewTile = () => {
      const tileValue = Math.random() > 0.85 ? 4 : 2;
      const randomEmptyCell = findEmptyCells()[Math.floor(Math.random() * findEmptyCells().length)];
      id.value += 1;
      tiles.value.push({
        x: randomEmptyCell.x,
        y: randomEmptyCell.y,
        value: tileValue,
        id: `mrdka${id.value}`
      });
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

    const moveTiles = (direction: Direction): void => {
      const axis = (direction === Direction.RIGHT || direction === Direction.LEFT) ? 'x' : 'y';
      const mood = (direction === Direction.RIGHT || direction === Direction.DOWN) ? 1 : -1; // positive or negative mood
      const limitPosition = (direction === Direction.RIGHT || direction === Direction.DOWN) ? 4 : 1; // positive or negative mood

      // console.log('axis: ', axis);
      // console.log('limitPosition: ', limitPosition);
      
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
          const isNextTileOccupied = !!sortedTiles[index - 1] && sortedTiles[index - 1][axis] === tile[axis] + mood;
          // let moveLenght = 4 - sortedTiles.length;
          // console.log('sortedTiles: ', sortedTiles);
          // console.log('isNextMergeable', isNextTileMergeable);
          // console.log('isNextTileOccupied', isNextTileOccupied);

          // if(index === 2) {
          //   console.log(sortedTiles[1]);
          // }
          console.log('Mergeable first: ' + isNextTileMergeable);
          console.log('Mergeable second: ' + isNextNextTileMergeable);
          if(isNextTileMergeable && !isNextNextTileMergeable) {
            tile[axis] = !!sortedTiles[index - 1] ? sortedTiles[index - 1][axis]: limitPosition;
            const tileToUpdateIndex = tiles.value.findIndex((target) => sortedTiles[index - 1].id === target.id);
            tiles.value[tileToUpdateIndex].value = tiles.value[tileToUpdateIndex].value * 2;
            tiles.value.splice(tiles.value.findIndex((target) => target.id === tile.id), 1);
          } 
          
          if((tile[axis] !== limitPosition && !isNextTileOccupied) || (isNextTileMergeable && isNextNextTileMergeable)) {
            tile[axis] = !!sortedTiles[index - 1] ? sortedTiles[index - 1][axis] - mood: limitPosition;
          } 
          // else if(isNextTileMergeable) {
          //   // Merge tiles
          //   setTimeout(() => {
          //     // const tileInRowToUpdateIndex = tilesInRow.value.findIndex((target) => sortedTiles[index - 1].id === target.id);
          //     const tileToUpdateIndex = tiles.value.findIndex((target) => sortedTiles[index - 1].id === target.id);
          //     tiles.value[tileToUpdateIndex].value = tiles.value[tileToUpdateIndex].value * 2;
          //     // tilesInRow.value.splice(tiles.value.findIndex((target) => target.id === tile.id), 1);
          //     tiles.value.splice(tiles.value.findIndex((target) => target.id === tile.id), 1);
          //   }, 100);
          //   // moveLenght += moveLenght;
          // }

          
          
        });
      }

      // setTimeout(() => generateNewTile(), 0);
      generateNewTile();
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
        <span style="display: block; font-size: 10px">{{tile.id}}</span>
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
  // transition: left 2s ease, top 2s ease;
}
</style>
