<template>
  <transition :css="false" @enter="enter" appear>
    <circle
      :cx="centerX"
      :cy="centerY"
      :r="radius"
      :fill="fillColor"
      :fill-opacity="opacity"
      :stroke="stockColor"
      :stroke-opacity="stockOpacity"
      stroke-width="5"
    />
  </transition>
</template>

<script lang="ts" setup>
import { ref, computed, defineEmits } from 'vue'
import { gsap, Bounce } from 'gsap'
import { OVER, HEXES } from '@/constants'

const props = defineProps<{
  checker: any
  cellSize: number
  rowCount: number
  radius: number
  status: string
}>()

const emit = defineEmits(['land'])

const minDuration = ref(0.2)
const coefficient = ref(0.4)

const row = computed(() => props.checker.row)
const col = computed(() => props.checker.col)
const centerX = computed(() => props.cellSize / 2)
const centerY = computed(
  () => props.cellSize / 2 + props.cellSize * (props.rowCount - 1 - row.value)
)
const fromY = computed(() => -1 * (centerY.value + props.cellSize))
const destY = computed(() => 0)
const isWinner = computed(() => props.checker.isWinner)
const color = computed(() => props.checker.color)
const fillColor = computed(() => HEXES[color.value])
const stockColor = computed(() => HEXES[color.value + 'Stock'])
const stockOpacity = computed(() => (props.status === OVER && !isWinner.value ? 0.3 : 1.0))
const opacity = computed(() => (props.status === OVER && !isWinner.value ? 0.3 : 1.0))
const percentage = computed(() => (props.rowCount - row.value) / props.rowCount)

const duration = () => {
  const percentage = (props.rowCount - row.value) / props.rowCount
  return minDuration.value + coefficient.value * percentage
}

const enter = (el: Element, done: () => void) => {
  const fromY = -1 * (centerY.value + props.cellSize)
  const destY = 0
  const fromParams = {
    y: fromY
  }
  const destParams = {
    y: destY,
    ease: Bounce.easeOut,
    onComplete: () => {
      emit('land')
      done()
    }
  }
  return gsap.fromTo(el, duration(), fromParams, destParams)
}
</script>
