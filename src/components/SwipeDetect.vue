<script lang="ts">
import { defineComponent, reactive } from 'vue';

export default defineComponent({
  name: 'SwipeDetect',
  props: {
    callback: {
      type: Function,
      required: true,
    },
  },

  setup(props) {
    // Source of knowledge http://www.javascriptkit.com/javatutors/touchevents2.shtml
    const swipeState = reactive({
      direction: 'none',
      startX: 0,
      startY: 0,
      distX: 0,
      distY: 0,
      threshold: 50, // Min distance traveled required to be considered as a swipe
      restraint: 100, // Max distance allowed in perpendicular direction
      allowedTime: 1000, // Maximum time allowed to travel that distance
      elapsedTime: 0,
      startTime: 0,
    });

    const clear = (): string => (swipeState.direction = 'none');

    const onTouchStart = (event: any): void => {
      const touch = event.changedTouches[0];
      swipeState.startX = touch.pageX;
      swipeState.startY = touch.pageY;
      swipeState.startTime = new Date().getTime(); // First finger contact
    };

    const onTouchEnd = (event: any): void => {
      const touch = event.changedTouches[0];
      swipeState.distX = touch.pageX - swipeState.startX;
      swipeState.distY = touch.pageY - swipeState.startY;
      swipeState.elapsedTime = new Date().getTime() - swipeState.startTime;

      if (swipeState.elapsedTime <= swipeState.allowedTime) {
        if (
          Math.abs(swipeState.distX) >= swipeState.threshold &&
          Math.abs(swipeState.distY) <= swipeState.restraint
        ) {
          // 2nd condition for horizontal swipe met
          swipeState.direction = swipeState.distX < 0 ? 'left' : 'right'; // if dist traveled is negative, it indicates left swipe
        } else if (
          Math.abs(swipeState.distY) >= swipeState.threshold &&
          Math.abs(swipeState.distX) <= swipeState.restraint
        ) {
          // 2nd condition for vertical swipe met
          swipeState.direction = swipeState.distY < 0 ? 'up' : 'down'; // if dist traveled is negative, it indicates up swipe
        }
      }
      props.callback(swipeState.direction);
      clear();
    };

    return { onTouchStart, onTouchEnd };
  },
});
</script>

<template>
  <div @touchstart="onTouchStart" @touchend="onTouchEnd">
    <slot />
  </div>
</template>
