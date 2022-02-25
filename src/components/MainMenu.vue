<template>
  <div
    class="mainmenu flex justify-center items-center flex-col m-3 p-2 text-center"
  >
    <div class="mainmenu__menu m-5 p-3">
      <ul class="mainmenu__menu-list flex flex-col gap-4">
        <li class="mainmenu__menu-list--item btn-game m-4" @click="newgame()">
          <span class="p-3">New Game</span>
        </li>
        <li
          class="mainmenu__menu-list--item btn-game m-4"
          @click="continue_game()"
        >
          <span class="p-3">Continue</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { defineEmits } from 'vue'

const emits = defineEmits(['newgame', 'continue_game'])

const newgame = () => {
  emits('newgame')
}

const continue_game = () => {
  emits('continue_game')
}
</script>

<style lang="scss" scoped>
.mainmenu {
  &__menu {
    &-list {
      perspective: 30rem;
      position: relative;
    }
  }
}
</style>
