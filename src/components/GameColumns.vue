<template>
  <svg :x="col * cellSize" y="0">
    <g @click="drop(col)" class="column">
      <GameCheckers
        v-for="checker in checkers"
        :key="key(checker)"
        :checker="checker"
        :cell-size="cellSize"
        :rowCount="rowCount"
        :radius="radius"
        :status="status"
        @land="land"
      />
      <rect
        :class="status"
        :key="col"
        :col="col"
        :width="cellSize"
        :height="boardHeight"
        :fill="color"
        :fill-opacity="opacity"
        :mask="mask"
      />
    </g>
  </svg>
</template>

<script lang="ts" setup>
import { computed, defineEmits } from 'vue'
import { OVER } from '@/constants'
import GameCheckers from '@/components/GameCheckers.vue'

const props = defineProps<{
  checkers: { row: number; col: number }[]
  col: number
  color: string
  cellSize: number
  boardHeight: number
  radius: number
  rowCount: number
  mask: string
  status: string
}>()

const emit = defineEmits(['land', 'drop'])

const nextEmptyRow = computed(() => {
  return Math.max(...props.checkers.map((c) => c.row).concat(-1)) + 1
})

const opacity = computed(() => {
  return props.status === OVER ? 0.25 : 1.0
})

const key = ({ row, col }: { row: number; col: number }) => {
  return `${row}${col}`
}

const land = () => {
  emit('land')
}

const drop = (col: number) => {
  const row = nextEmptyRow.value
  if (row < props.rowCount) {
    emit('drop', { row, col })
  }
}
</script>

<style scoped>
.column {
  cursor: pointer;
}
</style>
