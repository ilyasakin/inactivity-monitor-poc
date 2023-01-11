<template>
  <div id="app">
    <p>countdown: {{inactivityCountdown}}</p>
    <p>is mouse moving: {{isMouseMoving}}</p>
    <p>is scrolling: {{isScrolling}}</p>
    <p>is focused: {{isFocused}}</p>
    <p>is key pressing: {{isKeyPressing}}</p>
    <p>is touching: {{isTouching}}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, onBeforeMount, onBeforeUnmount } from "vue";

const MICRO_INACTIVITY_THRESHOLD = 1000;
const INACTIVITY_THRESHOLD = 60;

const inactivityCountdown = ref<number>(INACTIVITY_THRESHOLD);
const isMouseMoving = ref<boolean>(false);
const isMouseMovingTimeout = ref<NodeJS.Timeout | null>(null);
const isScrolling = ref<boolean>(false);
const isScrollingTimeout = ref<NodeJS.Timeout | null>(null);
const isFocused = ref<boolean>(false);
const isFocusedTimeout = ref<NodeJS.Timeout | null>(null);
const isKeyPressing = ref<boolean>(false);
const isKeyPressingTimeout = ref<NodeJS.Timeout | null>(null);
const isTouching = ref<boolean>(false);
const isTouchingTimeout = ref<NodeJS.Timeout | null>(null);

let inactivityInterval: NodeJS.Timeout;

const resetInactivityCountdown = () => {
  inactivityCountdown.value = INACTIVITY_THRESHOLD;
};

const onMouseMove = () => {
  resetInactivityCountdown();

  isMouseMoving.value = true;

  if (isMouseMovingTimeout.value) {
    clearTimeout(isMouseMovingTimeout.value);
    isMouseMovingTimeout.value = null;
  }

  isMouseMovingTimeout.value = setTimeout(() => {
    isMouseMoving.value = false;
    isMouseMovingTimeout.value = null;
  }, MICRO_INACTIVITY_THRESHOLD);
};

const onScroll = () => {
  resetInactivityCountdown();

  isScrolling.value = true;

  if (isScrollingTimeout.value) {
    clearTimeout(isScrollingTimeout.value);
    isScrollingTimeout.value = null;
  }

  isScrollingTimeout.value = setTimeout(() => {
    isScrolling.value = false;
    isScrollingTimeout.value = null;
  }, MICRO_INACTIVITY_THRESHOLD);
};

const onFocus = () => {
  resetInactivityCountdown();

  isFocused.value = true;

  if (isFocusedTimeout.value) {
    clearTimeout(isFocusedTimeout.value);
    isFocusedTimeout.value = null;
  }

  isFocusedTimeout.value = setTimeout(() => {
    isFocused.value = false;
    isFocusedTimeout.value = null;
  }, MICRO_INACTIVITY_THRESHOLD);
}

const onKeyPress = () => {
  resetInactivityCountdown();

  isKeyPressing.value = true;

  if (isKeyPressingTimeout.value) {
    clearTimeout(isKeyPressingTimeout.value);
    isKeyPressingTimeout.value = null;
  }

  isKeyPressingTimeout.value = setTimeout(() => {
    isKeyPressing.value = false;
    isKeyPressingTimeout.value = null;
  }, MICRO_INACTIVITY_THRESHOLD);
};

const onTouchStart = () => {
  resetInactivityCountdown();
  isTouching.value = true;
};

const onTouchMove = () => {
  resetInactivityCountdown();
};

const onTouchEnd = () => {
  resetInactivityCountdown();

  if (isTouchingTimeout.value) {
    clearTimeout(isTouchingTimeout.value);
    isTouchingTimeout.value = null;
  }

  isTouchingTimeout.value = setTimeout(() => {
    isTouching.value = false;
    isTouchingTimeout.value = null;
  }, MICRO_INACTIVITY_THRESHOLD);
};

const getIsInactivity = () => {
  return !isMouseMoving.value && !isScrolling.value && !isFocused.value && !isKeyPressing.value && !isTouching.value;
};

const handleInactivityCountdown = () => {
  if (!getIsInactivity() || inactivityCountdown.value <= 0) {
    return;
  }

  inactivityCountdown.value -= 1;
};

onBeforeMount(() => {
  document.addEventListener('mousemove', onMouseMove);
  document.addEventListener('focus', onFocus);
  document.addEventListener('keypress', onKeyPress)
  window.addEventListener('scroll', onScroll);
  document.addEventListener('touchstart', onTouchStart);
  document.addEventListener('touchmove', onTouchMove);
  document.addEventListener('touchend', onTouchEnd);
  inactivityInterval = setInterval(handleInactivityCountdown, 1000);
});

onBeforeUnmount(() => {
  document.removeEventListener('mousemove', onMouseMove);
  window.removeEventListener('scroll', onScroll);
  document.removeEventListener('focus', onFocus);
  document.removeEventListener('keypress', onKeyPress);
  document.removeEventListener('touchstart', onTouchStart);
  document.removeEventListener('touchmove', onTouchMove);
  document.removeEventListener('touchend', onTouchEnd);
  clearInterval(inactivityInterval);
});
</script>

<style scoped>
#app {
  font-family: monospace;
}
</style>
