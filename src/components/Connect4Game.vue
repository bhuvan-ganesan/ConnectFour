<template>
  <div class="connect4-game">
    <div class="board">
      <div
        v-for="(column, colIndex) in board"
        :key="colIndex"
        class="column"
        @click="dropDisk(colIndex)"
      >
        <div
          v-for="(cell, rowIndex) in column"
          :key="rowIndex"
          class="cell"
          :class="cellClass(cell)"
        ></div>
      </div>
    </div>
    <div class="status">
      <p v-if="winner">Player {{ winner }} wins!</p>
      <p v-else>Current Player: {{ currentPlayer }}</p>
    </div>
    <button @click="resetGame">Reset Game</button>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  setup() {
    const rows = 8
    const cols = 8
    const board = ref(Array.from({ length: cols }, () => Array(rows).fill(0)))
    const currentPlayer = ref(1)
    const winner = ref(null)

    const cellClass = (cell) => {
      return {
        empty: cell === 0,
        player1: cell === 1,
        player2: cell === 2
      }
    }

    const dropDisk = (colIndex) => {
      if (winner.value) return

      for (let rowIndex = rows - 1; rowIndex >= 0; rowIndex--) {
        if (board.value[colIndex][rowIndex] === 0) {
          board.value[colIndex][rowIndex] = currentPlayer.value
          if (checkWinner(colIndex, rowIndex)) {
            winner.value = currentPlayer.value
          } else {
            currentPlayer.value = currentPlayer.value === 1 ? 2 : 1
          }
          break
        }
      }
    }

    const checkWinner = (colIndex, rowIndex) => {
      return (
        checkDirection(colIndex, rowIndex, 1, 0) || // Horizontal
        checkDirection(colIndex, rowIndex, 0, 1) || // Vertical
        checkDirection(colIndex, rowIndex, 1, 1) || // Diagonal /
        checkDirection(colIndex, rowIndex, 1, -1) // Diagonal \
      )
    }

    const checkDirection = (colIndex, rowIndex, colDir, rowDir) => {
      let count = 0
      let player = board.value[colIndex][rowIndex]

      for (let i = -3; i <= 3; i++) {
        const col = colIndex + i * colDir
        const row = rowIndex + i * rowDir

        if (col >= 0 && col < cols && row >= 0 && row < rows && board.value[col][row] === player) {
          count++
          if (count === 4) return true
        } else {
          count = 0
        }
      }
      return false
    }

    const resetGame = () => {
      board.value = Array.from({ length: cols }, () => Array(rows).fill(0))
      currentPlayer.value = 1
      winner.value = null
    }

    return {
      board,
      currentPlayer,
      winner,
      cellClass,
      dropDisk,
      resetGame
    }
  }
}
</script>

<style scoped>
.connect4-game {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.board {
  display: flex;
}
.column {
  display: flex;
  flex-direction: column-reverse;
  margin: 2px;
}
.cell {
  width: 50px;
  height: 50px;
  margin: 1px;
  border-radius: 50%;
  background-color: lightgray;
  cursor: pointer;
}
.cell.empty {
  background-color: lightgray;
}
.cell.player1 {
  background-color: red;
}
.cell.player2 {
  background-color: yellow;
}
.status {
  margin-top: 20px;
}
</style>
