<template>
  <transition name="bounce">
    <div
      class="gamescreen__heading text-8xl tracking-wider font-weight-bold m-5 p-3"
      v-if="status !== 'startgame' && status !== 'continue_game'"
    >
      <h1>2048 Game</h1>
    </div>
  </transition>

  <transition name="bounce">
    <main-menu
      v-if="status === 'mainmenu'"
      @newgame="newgame"
      @continue_game="continue_game"
    />
  </transition>

  <transition name="bounce">
    <game-mode v-if="status === 'selectmode'" @set_gamemode="gamemode" />
  </transition>

  <transition name="bounce">
    <game-board
      v-if="status === 'startgame' || status === 'continue_game'"
      :rows="number_of_row"
      :cols="number_of_col"
      :continued="continued"
      @back_to_mainmenu="mainmenu"
    />
  </transition>
</template>

<script setup lang="ts">
import MainMenu from './MainMenu.vue'
import GameMode from './GameMode.vue'
import GameBoard from './GameBoard.vue'
import { ref } from 'vue'

const status = ref('mainmenu')
const number_of_row = ref(4)
const number_of_col = ref(4)
const continued = ref(false)

const newgame = () => {
  status.value = 'selectmode'
  continued.value = false
}

const continue_game = () => {
  status.value = 'continue_game'
  continued.value = true

  if (localStorage.getItem('gameboard')) {
    try {
      const store = JSON.parse(localStorage.getItem('gameboard'))

      number_of_row.value = store.rows
      number_of_col.value = store.cols
    } catch (error) {
      localStorage.removeItem('gameboard')
    }
  }
}

const gamemode = (rows: number, cols: number) => {
  status.value = 'startgame'
  number_of_row.value = rows
  number_of_col.value = cols
}

const mainmenu = () => {
  status.value = 'mainmenu'
}
</script>

<style lang="scss" scoped>
.gamescreen {
  &__heading {
    h1 {
      text-shadow: 0 0.1rem 0 rgba(134, 37, 37, 0.384),
        0 0.2rem 0 rgba(134, 37, 37, 0.384), 0 0.3rem 0 rgba(134, 37, 37, 0.384),
        0 0.4rem 0 rgba(134, 37, 37, 0.384), 0 0.5rem 0 rgba(134, 37, 37, 0.384),
        0 0.6rem 0 rgba(134, 37, 37, 0.384),
        0 5rem 3rem rgba(134, 37, 37, 0.384);
    }
  }
}

.bounce-enter-active {
  animation: bounce-in 0.5s;
}
.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    transform: scale(1.25);
    opacity: 0.5;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
</style>
