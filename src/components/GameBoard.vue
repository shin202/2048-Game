<template>
  <div
    class="main-screen flex justify-center items-center flex-col min-h-screen"
  >
    <game-score
      :score="score"
      @restart="restart_game"
      @back_to_home="back_to_home"
    />
    <div
      class="gameboard grid gap-2 p-4 rounded-lg"
      :class="`grid-rows-${rows} grid-cols-${cols}`"
    >
      <div
        class="gameboard__tiles rounded-lg flex justify-center items-center text-3xl"
        v-for="i in rows * cols"
        :key="i"
      >
        <div
          class="gameboard__tiles-number w-full h-full flex justify-center items-center"
          :class="`x${number[i - 1]} ${number[i - 1] !== 0 ? 'new' : ''}`"
        >
          <span v-if="number[i - 1] !== 0">{{ number[i - 1] }}</span>
        </div>
      </div>
    </div>
    <game-result v-if="over || win">
      <span v-if="over">Gameover</span>
      <div v-else class="flex justify-center items-center flex-col gap-4">
        <span>YouWin</span>
        <div
          class="btn-game text-xs w-60 flex justify-center items-center"
          @click="keep_running()"
        >
          <span>Continue?</span>
        </div>
      </div>
    </game-result>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, defineProps, defineEmits } from 'vue'
import GameScore from './GameScore.vue'
import GameResult from './GameResult.vue'

interface props {
  rows: number
  cols: number
  continued: boolean
}

const prop = defineProps<props>()
const { rows, cols, continued } = prop
const gameboard = ref([])
const number = ref([])
const score = ref(0)
const over = ref(false)
const win = ref(false)
const keeping = ref(false)

const create_gameboard = () => {
  score.value = 0
  over.value = false

  gameboard.value = new Array(rows).fill(0).map(() => {
    return new Array(cols).fill(0)
  })

  add_random_tile()
  add_random_tile()

  fill_number(gameboard.value)

  if (!continued) {
    set_local_storage(
      rows,
      cols,
      gameboard.value,
      number.value,
      score.value,
      over.value
    )
  } else {
    get_local_storage()
  }
}

const fill_number = (gameboard: number[]) => {
  number.value = []
  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < cols; col++) {
      number.value.push(gameboard[row][col])
    }
  }
}

const update_gameboard = (new_gameboard: number[]) => {
  gameboard.value = new_gameboard
  add_random_tile()
  fill_number(gameboard.value)

  set_local_storage(
    rows,
    cols,
    gameboard.value,
    number.value,
    score.value,
    over.value
  )

  moveable()
  if (!keeping.value) {
    winning(2048);
    remove_controller();
  } else {
    winning(99999);
    add_controller();
  }

  if (gameover()) {
    remove_controller()
    over.value = true
  }
}

// Save to local storage
const set_local_storage = (
  rows: number,
  cols: number,
  gameboard: number[],
  number: number[],
  score: number,
  over: boolean
) => {
  localStorage.setItem(
    'gameboard',
    JSON.stringify({
      rows: rows,
      cols: cols,
      gameboard: gameboard,
      number: number,
      score: score,
      over: over,
    })
  )
}

// Get data from local storage
const get_local_storage = () => {
  if (continued && localStorage.getItem('gameboard')) {
    try {
      const store = JSON.parse(localStorage.getItem('gameboard'))

      gameboard.value = store.gameboard
      number.value = store.number
      score.value = store.score
    } catch (error) {
      localStorage.removeItem('gameboard')
    }
  }
}

// Restart game
const restart_game = () => {
  score.value = 0
  over.value = false
  win.value = false
  keeping.value = false

  gameboard.value = new Array(rows).fill(0).map(() => {
    return new Array(cols).fill(0)
  })

  add_random_tile()
  add_random_tile()

  fill_number(gameboard.value)
  set_local_storage(
    rows,
    cols,
    gameboard.value,
    number.value,
    score.value,
    over.value
  )
  add_controller()
}

// Generate Random Tile
const random_tile = () => {
  let random_row: number = crypto.getRandomValues(new Uint32Array(1))[0]
  random_row = (random_row / rows) % (rows - 1)
  random_row = Math.floor(Math.round(random_row))

  let random_col: number = crypto.getRandomValues(new Uint32Array(1))[0]
  random_col = (random_col / cols) % (cols - 1)
  random_col = Math.floor(Math.round(random_col))

  return { row: random_row, col: random_col }
}

// Add tile to gameboard
const add_random_tile = () => {
  if (!empty_tile()) {
    return
  }

  const row = random_tile().row
  const col = random_tile().col

  if (gameboard.value[row][col] === 0) {
    gameboard.value[row][col] = 2
  } else {
    add_random_tile()
  }
}

// Create new array without zero
const filter_zero = (array: number[]): number[] => {
  return array.filter((number) => number !== 0)
}

const move = (array: number[]): number[] => {
  array = filter_zero(array)

  for (let i = 0; i < array.length - 1; i++) {
    if (array[i] === array[i + 1]) {
      array[i] *= 2
      score.value += array[i]
      array[i + 1] = 0
    }
  }

  array = filter_zero(array)

  while (array.length < cols) {
    array.push(0)
  }

  return array
}

const move_left = () => {
  const new_gameboard = []
  for (let row = 0; row < rows; row++) {
    let each_row: number[] = gameboard.value[row]
    each_row = move(each_row)
    gameboard.value[row] = each_row
    new_gameboard.push(gameboard.value[row])
  }

  update_gameboard(new_gameboard)
}

const move_right = () => {
  const new_gameboard = []
  for (let row = 0; row < rows; row++) {
    let each_row: number[] = gameboard.value[row]
    each_row.reverse()
    each_row = move(each_row)
    each_row.reverse()
    gameboard.value[row] = each_row
    new_gameboard.push(gameboard.value[row])
  }

  update_gameboard(new_gameboard)
}

const move_up = () => {
  for (let col = 0; col < cols; col++) {
    let each_col: number[] = gameboard.value.map((x) => {
      return x[col]
    })

    each_col = move(each_col)

    for (let row = 0; row < rows; row++) {
      gameboard.value[row][col] = each_col[row]
    }
  }

  update_gameboard(gameboard.value)
}

const move_down = () => {
  for (let col = 0; col < cols; col++) {
    let each_col: number[] = gameboard.value.map((x) => {
      return x[col]
    })
    each_col.reverse()
    each_col = move(each_col)
    each_col.reverse()

    for (let row = 0; row < rows; row++) {
      gameboard.value[row][col] = each_col[row]
    }
  }

  update_gameboard(gameboard.value)
}

// Controller
const control_left = (event: KeyboardEvent) => {
  if (event.key === 'ArrowLeft') {
    move_left()
  }
}
const control_right = (event: KeyboardEvent) => {
  if (event.key === 'ArrowRight') {
    move_right()
  }
}
const control_up = (event: KeyboardEvent) => {
  if (event.key === 'ArrowUp') {
    move_up()
  }
}
const control_down = (event: KeyboardEvent) => {
  if (event.key === 'ArrowDown') {
    move_down()
  }
}

const empty_tile = () => {
  let moveable = false
  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < cols; col++) {
      if (gameboard.value[row][col] === 0) {
        moveable = true
      }
    }
  }

  return moveable
}

// Check moveable left
const left_moveable = () => {
  let moveable = false

  for (let row = 0; row < rows; row++) {
    const each_row = gameboard.value[row]

    for (let i = 0; i < each_row.length - 1; i++) {
      const current_row_value = each_row[i]
      const next_row_value = each_row[i + 1]

      if (current_row_value === 0 && next_row_value !== 0) {
        moveable = true
      } else if (
        current_row_value !== 0 &&
        next_row_value !== 0 &&
        current_row_value === next_row_value
      ) {
        moveable = true
      }
    }
  }

  return moveable
}

// Check moveable right
const right_moveable = () => {
  let moveable = false

  for (let row = 0; row < rows; row++) {
    const each_row = gameboard.value[row]

    for (let i = 0; i < each_row.length - 1; i++) {
      const current_row_value = each_row[i]
      const next_row_value = each_row[i + 1]

      if (current_row_value !== 0 && next_row_value == 0) {
        moveable = true
      } else if (
        current_row_value !== 0 &&
        next_row_value !== 0 &&
        current_row_value === next_row_value
      ) {
        moveable = true
      }
    }
  }

  return moveable
}

const top_moveable = () => {
  let moveable = false

  for (let col = 0; col < cols; col++) {
    const each_col = gameboard.value.map((x) => {
      return x[col]
    })

    for (let i = 0; i < each_col.length - 1; i++) {
      const current_col_value = each_col[i]
      const next_col_value = each_col[i + 1]

      if (current_col_value === 0 && next_col_value !== 0) {
        moveable = true
      } else if (
        current_col_value !== 0 &&
        next_col_value !== 0 &&
        current_col_value === next_col_value
      ) {
        moveable = true
      }
    }
  }

  return moveable
}

const bottom_moveable = () => {
  let moveable = false

  for (let col = 0; col < cols; col++) {
    const each_col = gameboard.value.map((x) => {
      return x[col]
    })

    for (let i = 0; i < each_col.length - 1; i++) {
      const current_col_value = each_col[i]
      const next_col_value = each_col[i + 1]

      if (current_col_value !== 0 && next_col_value === 0) {
        moveable = true
      } else if (
        current_col_value !== 0 &&
        next_col_value !== 0 &&
        current_col_value === next_col_value
      ) {
        moveable = true
      }
    }
  }

  return moveable
}

const winning = (condition: number) => {
  if (number.value.includes(condition)) {
    win.value = true
  }
}

// Keep running when winning
const keep_running = () => {
  win.value = false
  keeping.value = true
}

const gameover = () => {
  let gameover = false

  if (
    !left_moveable() &&
    !right_moveable() &&
    !top_moveable() &&
    !bottom_moveable()
  ) {
    gameover = true
  }

  return gameover
}

const moveable = () => {
  // Left
  if (!left_moveable()) {
    document.removeEventListener('keyup', control_left)
  } else {
    document.addEventListener('keyup', control_left)
  }

  // Right
  if (!right_moveable()) {
    document.removeEventListener('keyup', control_right)
  } else {
    document.addEventListener('keyup', control_right)
  }

  // Up
  if (!top_moveable()) {
    document.removeEventListener('keyup', control_up)
  } else {
    document.addEventListener('keyup', control_up)
  }

  // Down
  if (!bottom_moveable()) {
    document.removeEventListener('keyup', control_down)
  } else {
    document.addEventListener('keyup', control_down)
  }
}

const add_controller = () => {
  document.addEventListener('keyup', control_left)
  document.addEventListener('keyup', control_right)
  document.addEventListener('keyup', control_up)
  document.addEventListener('keyup', control_down)
}

const remove_controller = () => {
  document.removeEventListener('keyup', control_left)
  document.removeEventListener('keyup', control_right)
  document.removeEventListener('keyup', control_up)
  document.removeEventListener('keyup', control_down)
}

// Go back to mainmenu
const emits = defineEmits(['back_to_mainmenu'])
const back_to_home = () => {
  emits('back_to_mainmenu')
}

onMounted(() => {
  create_gameboard()
  add_controller()
})
</script>

<style lang="scss" scoped>
.main-screen {
  width: clamp(30rem, 90%, 72rem);

  .gameboard {
    width: clamp(30rem, 90%, 55rem);
    min-height: 55rem;
    background-color: rgb(254, 242, 242);
    box-shadow: 2rem 2rem 4rem rgba(241, 104, 104, 0.25),
      -2rem -2rem 4rem rgba(241, 104, 104, 0.15),
      inset -0.5rem -0.5rem 2rem rgba(241, 104, 104, 0.25);

    &__tiles {
      border: 0.1rem solid rgba(0, 0, 0, 0.2);
      background-color: rgb(248, 228, 228);
      overflow: hidden;

      &-number {
        transition: all 0.33s cubic-bezier(0.1, 0.68, 0.93, 0.73);
        background-color: rgb(248, 228, 228);

        &.new {
          animation: jello-horizontal 1.5s cubic-bezier(0.1, 0.68, 0.93, 0.73)
            both 1;
        }

        &.x0 {
          background-color: rgb(248, 228, 228);
        }
        &.x2 {
          background-color: rgb(247, 192, 192);
        }
        &.x4 {
          background-color: rgb(247, 162, 162);
        }
        &.x8 {
          background-color: rgb(245, 135, 135);
        }
        &.x16 {
          background-color: rgb(255, 118, 118);
        }
        &.x32 {
          background-color: rgb(245, 99, 99);
        }
        &.x64 {
          background-color: rgb(247, 86, 86);
        }
        &.x128 {
          background-color: rgb(250, 71, 71);
        }
        &.x256 {
          background-color: rgb(250, 128, 71);
        }
        &.x512 {
          background-color: rgb(71, 250, 116);
        }
        &.x1024 {
          background-color: rgb(71, 250, 250);
        }
        &.x2048 {
          background-color: rgb(214, 230, 78);
        }
        &.x4096 {
          background-color: rgb(206, 63, 87);
        }
      }

      @keyframes jello-horizontal {
        0% {
          -webkit-transform: scale3d(0, 0, 0);
          transform: scale3d(0, 0, 0);
        }
        15% {
          -webkit-transform: scale3d(1, 1, 1);
          transform: scale3d(1, 1, 1);
        }
        30% {
          -webkit-transform: scale3d(1.25, 0.75, 1);
          transform: scale3d(1.25, 0.75, 1);
        }
        40% {
          -webkit-transform: scale3d(0.75, 1.25, 1);
          transform: scale3d(0.75, 1.25, 1);
        }
        50% {
          -webkit-transform: scale3d(1.15, 0.85, 1);
          transform: scale3d(1.15, 0.85, 1);
        }
        65% {
          -webkit-transform: scale3d(0.95, 1.05, 1);
          transform: scale3d(0.95, 1.05, 1);
        }
        75% {
          -webkit-transform: scale3d(1.05, 0.95, 1);
          transform: scale3d(1.05, 0.95, 1);
        }
        100% {
          -webkit-transform: scale3d(1, 1, 1);
          transform: scale3d(1, 1, 1);
        }
      }
    }
  }
}
</style>
