<template>
  <div class="gamemode flex flex-col justify-center items-center m-3 p-2">
    <h1 class="m-5 p-3">Select Mode</h1>
    <div class="gamemode__mode">
      <ul class="gamemode__mode-list flex gap-4">
        <li
          class="gamemode__mode-list--item btn-game m-4"
          v-for="(mode, index) in modes"
          :key="index"
          @click="set_gamemode(mode.rows, mode.cols)"
        >
          <span class="p-3">{{ mode.name }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, defineEmits } from 'vue'

interface gamemode {
  name?: string
  rows: number
  cols: number
}

const modes: Array<gamemode> = reactive([
  {
    name: '4x4',
    rows: 4,
    cols: 4,
  },
  {
    name: '5x5',
    rows: 5,
    cols: 5,
  },
  {
    name: '6x6',
    rows: 6,
    cols: 6,
  },
  {
    name: '8x8',
    rows: 8,
    cols: 8,
  },
])

const emits = defineEmits(['set_gamemode'])
const set_gamemode = (rows: number, cols: number) => {
  emits('set_gamemode', rows, cols)
}
</script>

<style lang="scss" scoped>
.gamemode {
  &__mode {
    &-list {
      &--item {
        text-align: center;
      }
    }
  }
}
</style>
