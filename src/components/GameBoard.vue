<template>
  <svg
    :viewBox="`0 0 ${boardWidth} ${boardHeight}`"
    xmlns="http://www.w3.org/2000/svg"
    class="game-board"
    :style="{ 'border-color': borderColor }"
  >
    <defs>
      <pattern :id="patternId" patternUnits="userSpaceOnUse" :width="cellSize" :height="cellSize">
        <circle :cx="cellSize / 2" :cy="cellSize / 2" :r="radius" fill="black" />
      </pattern>
      <mask :id="maskId">
        <rect :width="cellSize" :height="boardHeight" fill="white" />
        <rect :width="cellSize" :height="boardHeight" :fill="pattern" />
      </mask>
    </defs>
    <GameColumns
      v-for="col in cols"
      :key="col"
      :checkers="colCheckers(col)"
      :col="col"
      :color="columnsColor"
      :cell-size="cellSize"
      :radius="radius"
      :board-height="boardHeight"
      :row-count="rowCount"
      :mask="mask"
      :status="status"
      @drop="drop"
      @land="land"
    />
  </svg>
</template>

<script lang="ts" setup>
import { ref, computed, defineEmits } from 'vue'
import { cssUrl, range } from '@/utils.js'
import GameColumns from '@/components/GameColumns.vue'
import { HEXES } from '@/constants'

const props = defineProps<{
  checkers: Record<string, { col: number; row: number }>
  rowCount: number
  colCount: number
  status: string
  currentPlayer: string
}>()

const emit = defineEmits(['drop', 'land'])

const patternId = 'cell-pattern'
const maskId = 'cell-mask'
const cellSize = 100
const borderColor = computed(() => HEXES[props.currentPlayer])
const pattern = computed(() => cssUrl(patternId))
const mask = computed(() => cssUrl(maskId))
const rows = computed(() => range(props.rowCount))
const cols = computed(() => range(props.colCount))
const boardWidth = computed(() => props.colCount * cellSize)
const boardHeight = computed(() => props.rowCount * cellSize)
const radius = computed(() => cellSize * 0.45)
const columnsColor = computed(() => HEXES.columns)

function colCheckers(col: number) {
  return Object.values(props.checkers)
    .filter((c) => c.col === col)
    .sort((a, b) => a.row - b.row)
}

function drop(data: any) {
  emit('drop', data)
}

function land() {
  emit('land')
}
</script>

<style scoped>
.game-board {
  border: 5px solid;
  margin: 0 auto;
  width: 80%;
  max-width: 420px;
}
</style>
