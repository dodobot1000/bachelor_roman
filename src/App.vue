<script setup>
import { ref } from 'vue'
import GameIntro from './components/GameIntro.vue'
import LevelOne from './components/LevelOne.vue'
import LevelTwo from './components/LevelTwo.vue'
import LevelThree from './components/LevelThree.vue'
import GameOver from './components/GameOver.vue'
import GameWin from './components/GameWin.vue'

const scene = ref('intro')
const currentLevel = ref(1)
const winMessage = ref('')

const levelScenes = { 1: 'level1', 2: 'level2', 3: 'level3' }

function goToLevel() {
  currentLevel.value = 1
  scene.value = 'level1'
}

function onLose() {
  scene.value = 'gameover'
}

function onWinLevel1() {
  winMessage.value = 'Tu as regard\u00e9 tout le vid\u00e9o sans te faire prendre!'
  currentLevel.value = 1
  scene.value = 'win'
}

function onWinLevel2() {
  winMessage.value = 'Tu connais bien tes substances!'
  currentLevel.value = 2
  scene.value = 'win'
}

function onWinLevel3() {
  winMessage.value = 'Tu as fui la favela! Les gars sont saufs!'
  currentLevel.value = 3
  scene.value = 'win'
}

function retry() {
  scene.value = levelScenes[currentLevel.value]
}

function nextLevel() {
  const next = currentLevel.value + 1
  if (next <= 3) {
    currentLevel.value = next
    scene.value = levelScenes[next]
  }
}
</script>

<template>
  <Transition name="fade" mode="out-in">
    <GameIntro v-if="scene === 'intro'" @play="goToLevel" />
    <LevelOne v-else-if="scene === 'level1'" @lose="onLose" @win="onWinLevel1" />
    <LevelTwo v-else-if="scene === 'level2'" @lose="onLose" @win="onWinLevel2" />
    <LevelThree v-else-if="scene === 'level3'" @lose="onLose" @win="onWinLevel3" />
    <GameOver v-else-if="scene === 'gameover'" @retry="retry" />
    <GameWin
      v-else-if="scene === 'win'"
      :message="winMessage"
      :has-next="currentLevel < 3"
      @next="nextLevel"
    />
  </Transition>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
