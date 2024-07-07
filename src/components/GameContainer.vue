<template>
  <div>
    <p>To play, click on any of the columns.!</p>
    <GameScoreBoard
      :moves="moves"
      :winner="winner"
      :current-player="currentPlayer"
      :status="status"
      @reset="reset"
    />
    <GameBoard
      :current-player="currentPlayer"
      :checkers="checkers"
      :row-count="rowCount"
      :col-count="colCount"
      :status="status"
      @drop="drop"
      @land="land"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { min, max, key } from '@/utils.js'
import { AI, YOU, EMPTY, PLAY, OVER } from '@/constants'
import GameBoard from '@/components/GameBoard.vue'
import GameScoreBoard from '@/components/GameScoreBoard.vue'
import Position from '@/utilities/Position'
import Solver from '@/utilities/Solver'

const WIDTH = 7
const HEIGHT = 6

interface Checker {
  row: number
  col: number
  color?: string
  isWinner?: boolean
}

const props = defineProps<{ firstPlayer: string }>()

const checkers = ref<Record<string, Checker>>({})
const isLocked = ref(false)
const rowCount = ref(HEIGHT)
const colCount = ref(WIDTH)
const status = ref(PLAY)
const currentPlayer = ref(props.firstPlayer)
const winner = ref(undefined)
const position = ref(new Position(WIDTH, HEIGHT))
const solver = ref(new Solver(WIDTH))
const counter = ref(0)

const moves = computed(() => Object.values(checkers.value))
const gameOver = computed(() => status.value === OVER)
const isDraw = computed(
  () => Object.keys(checkers.value).length === rowCount.value * colCount.value
)
// const currentPlayer = computed(() => props.firstPlayer)

onMounted(() => {
  if (currentPlayer.value === AI) {
    const scores = solver.value.analyze(position.value)
    let bestMove = 3
    let bestScore = position.value.minScore - 2
    for (let col = 0; col < position.value.width; col++) {
      const orderedCol = solver.value.columnExpOrderRev[col]
      if (scores[orderedCol] >= bestScore) {
        bestScore = scores[orderedCol]
        bestMove = orderedCol
      }
    }
    const colCheckers = Object.values(checkers.value)
      .filter((c) => c.col === bestMove)
      .sort((a, b) => a.row - b.row)
    const lastRow = Math.max(...colCheckers.map((c) => c.row).concat(-1)) + 1
    drop({ col: bestMove, row: lastRow })
  }
})

function reset() {
  window.history.go(0)
}

function toggleColor() {
  if (currentPlayer.value === AI) {
    currentPlayer.value = YOU
  } else {
    currentPlayer.value = AI
  }
}

function getChecker({ row, col }: { row: number; col: number }) {
  return checkers.value[key(row, col)] || { row, col, color: 'empty' }
}

function setChecker({ row, col }: { row: number; col: number }, attrs = {}) {
  const checker = getChecker({ row, col })
  checkers.value[key(row, col)] = { ...checker, ...attrs }
}

function drop({ col, row }: { col: number; row: number }) {
  if (isLocked.value) return
  isLocked.value = true
  const color = currentPlayer.value
  setChecker({ row, col }, { color })
  position.value.playCol(col)
  if (!isDraw.value) checkForWinFrom({ row, col })
}

function land() {
  if (isDraw.value) return displayDraw()
  if (winner.value) {
    displayWin(winner.value)
  } else {
    isLocked.value = false
    toggleColor()
  }
  if (currentPlayer.value === AI && !winner.value) {
    let scores
    if (counter.value <= 6) scores = solver.value.analyze(position.value, true)
    else scores = solver.value.analyze(position.value)
    counter.value++

    let bestMove = 3
    let bestScore = position.value.minScore - 2

    for (let col = 0; col < position.value.width; col++) {
      const orderedCol = solver.value.columnExpOrderRev[col]
      if (scores[orderedCol] >= bestScore) {
        bestScore = scores[orderedCol]
        bestMove = orderedCol
      }
    }

    const colCheckers = Object.values(checkers.value)
      .filter((c) => c.col === bestMove)
      .sort((a, b) => a.row - b.row)
    const lastRow = Math.max(...colCheckers.map((c) => c.row).concat(-1)) + 1
    drop({ col: bestMove, row: lastRow })
  }
}

function getWinner(...segment: [number, number][]) {
  if (segment.length !== 4) return false
  const checkersSegment = segment.map(([row, col]) => getChecker({ row, col }))
  const [{ color }] = checkersSegment
  if (color === EMPTY) return false
  return checkersSegment.every((c) => c.color === color) && { color, checkers: checkersSegment }
}

function checkHorizontalSegments({
  focalRow,
  minCol,
  maxCol
}: {
  focalRow: number
  minCol: number
  maxCol: number
}) {
  let winner
  for (let row = focalRow, col = minCol; col <= maxCol; col++) {
    winner = getWinner([row, col], [row, col + 1], [row, col + 2], [row, col + 3])
    if (winner) return winner
  }
}

function checkVerticalSegments({
  focalRow,
  focalCol,
  minRow
}: {
  focalRow: number
  focalCol: number
  minRow: number
}) {
  let winner
  for (let col = focalCol, row = minRow; row <= focalRow; row++) {
    winner = getWinner([row, col], [row + 1, col], [row + 2, col], [row + 3, col])
    if (winner) return winner
  }
}

function checkForwardSlashSegments({ focalRow, focalCol, minRow, minCol, maxRow, maxCol }: any) {
  const startForwardSlash = (row: number, col: number) => {
    while (row > minRow && col > minCol) {
      row--
      col--
    }
    return [row, col]
  }
  let winner
  for (
    let [row, col] = startForwardSlash(focalRow, focalCol);
    row <= maxRow && col <= maxCol;
    row++, col++
  ) {
    winner = getWinner([row, col], [row + 1, col + 1], [row + 2, col + 2], [row + 3, col + 3])
    if (winner) return winner
  }
}

function checkBackwardSlashSegments({ focalRow, focalCol, minRow, minCol, maxRow, maxCol }: any) {
  const startBackwardSlash = (row: number, col: number) => {
    while (row < maxRow && col > minCol) {
      row++
      col--
    }
    return [row, col]
  }
  let winner
  for (
    let [row, col] = startBackwardSlash(focalRow, focalCol);
    row >= minRow && col <= maxCol;
    row--, col++
  ) {
    winner = getWinner([row, col], [row - 1, col + 1], [row - 2, col + 2], [row - 3, col + 3])
    if (winner) return winner
  }
}

function checkForWinFrom(lastChecker: Checker) {
  if (!lastChecker) return
  const { row: focalRow, col: focalCol } = lastChecker
  const minCol = min(focalCol)
  const maxCol = max(focalCol, colCount.value - 1)
  const minRow = min(focalRow)
  const maxRow = max(focalRow, rowCount.value - 1)
  const coords = { focalRow, focalCol, minRow, minCol, maxRow, maxCol }
  winner.value =
    checkHorizontalSegments(coords) ||
    checkVerticalSegments(coords) ||
    checkForwardSlashSegments(coords) ||
    checkBackwardSlashSegments(coords)
}

function displayDraw() {
  status.value = OVER
}

function displayWin(winner) {
  status.value = OVER
  winner.checkers.forEach((checker) => {
    setChecker(checker, { isWinner: true })
  })
}

// function resigned(player) {
//   winner.value = { color: player === AI ? YOU : AI }
//   status.value = OVER
// }
</script>
