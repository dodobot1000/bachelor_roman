<script setup>
import { ref } from 'vue'
import GameIntro from './components/GameIntro.vue'
import LevelOne from './components/LevelOne.vue'
import LevelTwo from './components/LevelTwo.vue'
import LevelThree from './components/LevelThree.vue'
import LevelFour from './components/LevelFour.vue'
import GameOver from './components/GameOver.vue'
import GameWin from './components/GameWin.vue'

const scene = ref('intro')
const currentLevel = ref(1)
const winMessage = ref('')

const MAX_LEVEL = 4
const levelScenes = { 1: 'level1', 2: 'level2', 3: 'level3', 4: 'level4' }

function goToLevel() {
  currentLevel.value = 1
  scene.value = 'level1'
}

function selectLevel(level) {
  currentLevel.value = level
  scene.value = levelScenes[level]
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

function onWinLevel4() {
  winMessage.value = 'Tu connais bien l\'\u00e2ge des br\u00e9siliennes!'
  currentLevel.value = 4
  scene.value = 'win'
}

function goToMenu() {
  scene.value = 'intro'
}

function retry() {
  scene.value = levelScenes[currentLevel.value]
}

function nextLevel() {
  const next = currentLevel.value + 1
  if (next <= MAX_LEVEL) {
    currentLevel.value = next
    scene.value = levelScenes[next]
  }
}
</script>

<template>
  <Transition name="fade" mode="out-in">
    <GameIntro v-if="scene === 'intro'" @play="goToLevel" @select-level="selectLevel" />
    <LevelOne v-else-if="scene === 'level1'" @lose="onLose" @win="onWinLevel1" @menu="goToMenu" />
    <LevelTwo v-else-if="scene === 'level2'" @lose="onLose" @win="onWinLevel2" @menu="goToMenu" />
    <LevelThree v-else-if="scene === 'level3'" @lose="onLose" @win="onWinLevel3" @menu="goToMenu" />
    <LevelFour v-else-if="scene === 'level4'" @lose="onLose" @win="onWinLevel4" @menu="goToMenu" />
    <GameOver v-else-if="scene === 'gameover'" @retry="retry" @menu="goToMenu" />
    <GameWin
      v-else-if="scene === 'win'"
      :message="winMessage"
      :has-next="currentLevel < MAX_LEVEL"
      @next="nextLevel"
      @menu="goToMenu"
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
