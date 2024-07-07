<template>
  <div class="score-board">
    <div class="p1 cell">
      <p :style="{ color: aiHex }">Computer</p>
    </div>
    <div class="status cell">
      <p v-if="gameOver">
        {{ messageOver }}
        <br /><a class="play" href="#app" @click="reset">Play Again</a>
      </p>
      <p v-else>
        {{ currentPlayerTurnString }}
      </p>
    </div>
    <div class="p2 cell">
      <p :style="{ color: youHex }">You</p>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed, defineEmits } from 'vue'
import { HEXES, YOU, OVER } from '@/constants'

const props = defineProps<{
  winner: any
  moves: any[]
  currentPlayer: string
  status: string
}>()

const emit = defineEmits<{
  (event: 'reset'): void
}>()
const aiHex = ref(HEXES.ai)
const youHex = ref(HEXES.you)

const gameOver = computed(() => props.status === OVER)

const messageOver = computed(() => {
  if (props.winner && props.winner.color === 'ai') {
    return `Computer wins!`
  } else if (props.winner && props.winner.color === 'you') {
    return `You wins!`
  }
  return "It's a tie!"
})

const currentPlayerTurnString = computed(() => {
  if (props.currentPlayer === YOU) {
    return "It's your time now."
  } else {
    return "The computer's turn"
  }
})

function reset() {
  emit('reset')
}
</script>

<style scoped>
.play {
  color: #cf6679;
}
.score-board {
  width: 410px;
  display: grid;
  grid-template-columns: [p1] 25% [status] 50% [p2] 25%;
  margin: 0 auto;
  padding: 1em 0;
  font-weight: bold;
  font-size: large;
}
.score-board .status {
  font-weight: normal;
}
</style>
