<template>
  <div class="gamescore m-3 p-2 flex justify-around w-full">
    <div class="gamescore__score flex justify-center items-center flex-col">
      <div class="gamescore__score--title">
        <span>Score</span>
      </div>
      <div class="gamescore__score--score">
        <span>{{ score }}</span>
      </div>
    </div>
    <div class="gamescore__menu">
      <ul class="gamescore__menu--list flex justify-center items-center gap-3">
        <li
          class="gamescore__menu--list__item w-12 h-12 cursor-pointer"
          @click="back_to_home()"
        >
          <home-icon />
        </li>
        <li
          class="gamescore__menu--list__item w-12 h-12 cursor-pointer"
          @click="restart()"
        >
          <refresh-icon />
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { HomeIcon, RefreshIcon } from '@heroicons/vue/outline'
import { defineProps, defineEmits } from 'vue'

interface props {
  score: number
}

const prop = defineProps<props>()
const emits = defineEmits(['restart', 'back_to_home'])

const restart = () => {
  emits('restart')
}
const back_to_home = () => {
  emits('back_to_home')
}
</script>

<style lang="scss" scoped>
.gamescore {
  z-index: 99;
}
</style>
