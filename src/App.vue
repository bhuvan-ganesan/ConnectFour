<template>
  <div id="app">
    <div class="logo">
      <img alt="Connect 4 AI" src="@/assets/connect4Logo.png" />
    </div>
    <div v-if="isReady" id="game">
      <component :is="GameContainer" v-bind="gameContainerProps" />
    </div>
    <div v-else>
      <h3>Who wants to play first ?</h3>
      <button class="btn ai" type="button" @click="setPlayer('ai')">Computer</button>
      <button class="btn me" type="button" @click="setPlayer('you')">You</button>
      <br />
      <br />
    </div>
    <footer>
      Connect 4 |
      <a href="https://pixelscripts.in/" target="_blank">Developed by Pixel Scripts Team</a>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import GameContainer from '@/components/GameContainer.vue'
import { AI, YOU, HEXES } from '@/constants'

const currentPlayer = ref(undefined)
const isReady = ref(false)
const prompt = ref(false)

// const gameContainerName = computed(() => 'GameContainer')
const gameContainerProps = computed(() => ({ firstPlayer: currentPlayer.value }))

const setPlayer = (player) => {
  if (player === 'ai') currentPlayer.value = AI
  else if (player === 'you') currentPlayer.value = YOU
  if (currentPlayer.value) isReady.value = true
}

const update = async () => {
  prompt.value = false
  await window.$workbox.messageSW({ type: 'SKIP_WAITING' })
}

onMounted(() => {
  if (window.$workbox) {
    window.$workbox.addEventListener('waiting', () => {
      prompt.value = true
    })
  }
})
</script>

<style>
body {
  background-color: #121212;
}
a {
  text-decoration: none;
}
.logo {
  display: inline-block;
}
.logo h3 {
  margin: 0;
}
.btn {
  background-color: #5016e6;
  border-radius: 6px;
  padding: 12px 24px;
  transition-timing-function: ease-in;
  transition-property: background-color;
  transition-duration: 100ms;
  margin: 8px;
  font-size: large;
}
.ai {
  background-color: v-bind('HEXES.ai');
  color: #ffffff;
}
.me {
  background-color: v-bind('HEXES.you');
  color: #ffffff;
}
.ai:hover {
  background-color: v-bind('HEXES.aiStock');
}
.me:hover {
  background-color: v-bind('HEXES.youStock');
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #e6e6e7;
  margin-top: 60px;
}
</style>
