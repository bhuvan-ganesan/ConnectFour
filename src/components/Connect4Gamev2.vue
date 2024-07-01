<template>
  <div>
    <button @click="promptNewGame">{{ newGameText }}</button>
    <button @click="setAIDifficulty">{{ setAIText }}</button>
    <button @click="showAbout">{{ aboutText }}</button>
    <Connect4Board :board="board" @dropPiece="dropPiece" />
    <p>{{ statusMessage }}</p>
  </div>
</template>

<script>
import { ref, reactive } from 'vue'
import Connect4Board from './Connect4Board.vue'

export default {
  components: {
    Connect4Board
  },
  setup() {
    const newGameText = 'New Game'
    const setAIText = 'Set AI Difficulty'
    const aboutText = 'About'
    const statusMessage = ref('Your turn human.')
    const board = reactive(
      Array(8)
        .fill()
        .map(() => Array(8).fill('EMPTY'))
    )
    const HUMAN = 'HUMAN'
    const COMPUTER = 'COMPUTER'
    const EMPTY = 'EMPTY'
    const AI_LEVELS = { EASY: 3, MEDIUM: 6, HARD: 8 }
    const AI_RAND_WEIGHTS = { EASY: 1.0, MEDIUM: 0.5, HARD: 0.1 }
    let currentTurn = ref(HUMAN)
    let aiDifficulty = ref('MEDIUM')

    const promptNewGame = () => {
      resetBoard()
      currentTurn.value = HUMAN
      statusMessage.value = 'Your turn human.'
    }

    const setAIDifficulty = () => {
      const level = prompt('Choose AI Difficulty: EASY, MEDIUM, HARD', 'MEDIUM')
      if (level) aiDifficulty.value = level
    }

    const showAbout = () => {
      alert('Connect Four was written by Jared Ostmeyer')
    }

    const dropPiece = (column) => {
      if (currentTurn.value === HUMAN) {
        for (let row = 7; row >= 0; row--) {
          if (board[column][row] === EMPTY) {
            board[column][row] = HUMAN
            if (checkWin(column, row, HUMAN)) {
              statusMessage.value = 'The human has won!'
            } else {
              currentTurn.value = COMPUTER
              statusMessage.value = 'The computer is thinking.'
              setTimeout(makeAIMove, 1000)
            }
            break
          }
        }
      }
    }

    const makeAIMove = () => {
      const ply = AI_LEVELS[aiDifficulty.value]
      const randWeight = AI_RAND_WEIGHTS[aiDifficulty.value]
      const move = findOptimalMove(COMPUTER, ply, randWeight)
      if (move !== -1) {
        for (let row = 7; row >= 0; row--) {
          if (board[move][row] === EMPTY) {
            board[move][row] = COMPUTER
            if (checkWin(move, row, COMPUTER)) {
              statusMessage.value = 'The computer has won!'
            } else {
              statusMessage.value = 'Your turn human.'
              currentTurn.value = HUMAN
            }
            break
          }
        }
      }
    }

    const resetBoard = () => {
      for (let x = 0; x < 8; x++) {
        for (let y = 0; y < 8; y++) {
          board[x][y] = EMPTY
        }
      }
    }

    const checkWin = (column, row, player) => {
      return (
        checkDirection(column, row, player, 1, 0) || // Horizontal
        checkDirection(column, row, player, 0, 1) || // Vertical
        checkDirection(column, row, player, 1, 1) || // Diagonal /
        checkDirection(column, row, player, 1, -1)
      ) // Diagonal \
    }

    const checkDirection = (column, row, player, deltaX, deltaY) => {
      let count = 0
      for (let i = -3; i <= 3; i++) {
        const x = column + i * deltaX
        const y = row + i * deltaY
        if (x >= 0 && x < 8 && y >= 0 && y < 8 && board[x][y] === player) {
          count++
          if (count === 4) return true
        } else {
          count = 0
        }
      }
      return false
    }

    const findOptimalMove = (player, ply, randWeight) => {
      const moves = []
      for (let x = 0; x < 8; x++) {
        if (board[x][0] === EMPTY) {
          moves.push({ column: x, score: scoreMove(x, player, ply, randWeight) })
        }
      }
      moves.sort((a, b) => b.score - a.score)
      return moves.length > 0 ? moves[0].column : -1
    }

    const scoreMove = (column, player, ply, randWeight) => {
      let score = 0
      for (let row = 7; row >= 0; row--) {
        if (board[column][row] === EMPTY) {
          board[column][row] = player
          if (checkWin(column, row, player)) {
            score = player === COMPUTER ? 1000 : -1000
          } else if (ply > 1) {
            score +=
              scoreMove(
                findOptimalMove(-player, ply - 1, randWeight),
                -player,
                ply - 1,
                randWeight
              ) / 8.0
          }
          board[column][row] = EMPTY
          break
        }
      }
      return score + (Math.random() - 0.5) * randWeight
    }

    return {
      newGameText,
      setAIText,
      aboutText,
      statusMessage,
      board,
      promptNewGame,
      setAIDifficulty,
      showAbout,
      dropPiece
    }
  }
}
</script>

<style scoped>
/* Add your styles here */
</style>
