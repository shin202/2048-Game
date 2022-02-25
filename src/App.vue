<template>
  <game-screen />
</template>

<script setup lang="ts">
import GameScreen from './components/GameScreen.vue'
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+Mono&family=Press+Start+2P&display=swap');
@tailwind base;
@tailwind components;
@tailwind utilities;

html {
  font-size: 62.5%;
}

#app {
  font-size: 1.6rem;
  font-family: 'Press Start 2P', cursive;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: rgb(254, 242, 242);
  color: #690c0c;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  user-select: none;
  overflow: hidden;
}

@layer components {
  .btn-game {
    @apply py-2 px-2 rounded;
    background-color: #f9c4a6;
    min-width: 10rem;
    min-height: 5rem;
    line-height: 4;
    cursor: pointer;
    transform-style: preserve-3d;
    transition: all 1s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    box-shadow: 0.1rem 0.1rem rgba(238, 140, 49, 0.473),
      0.2rem 0.2rem rgba(238, 140, 49, 0.473),
      0.3rem 0.3rem rgba(238, 140, 49, 0.473),
      0.4rem 0.4rem rgba(238, 140, 49, 0.473),
      0 4rem 4rem rgba(238, 140, 49, 0.473),
      inset -0.2rem -0.2rem 0.4rem rgba(238, 140, 49, 0.473),
      inset 0.2rem 0.2rem 1rem rgba(238, 140, 49, 0.5);

    &:hover {
      transform: translateX(-2rem) translateZ(3rem);
      box-shadow: 0.2rem 0.2rem rgba(238, 140, 49, 0.473),
        0.4rem 0.4rem rgba(238, 140, 49, 0.473),
        0.6rem 0.6rem rgba(238, 140, 49, 0.473),
        0.8rem 0.8rem rgba(238, 140, 49, 0.473),
        inset -0.4rem -0.4rem 0.8rem rgba(238, 140, 49, 0.473),
        inset 0.2rem 0.2rem 0.8rem rgba(238, 140, 49, 0.473),
        0 4rem 4rem rgba(238, 140, 49, 0.473);
    }
  }
}
</style>
