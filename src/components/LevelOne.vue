<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['lose', 'win', 'menu'])

// Game states: 'intro', 'playing'
const gameState = ref('intro')

// Watching state
const isHolding = ref(false)
const watchedTime = ref(0)
const TOTAL_TIME = 15

// Cintia states: 'back', 'turning', 'facing'
const cintiaState = ref('back')
const showWarning = ref(false)

// Intervals and timers
let watchInterval = null
let cintiaTimeout = null
let turningTimeout = null
let facingTimeout = null
let returnTimeout = null

const progress = computed(() => (watchedTime.value / TOTAL_TIME) * 100)

const cintiaImage = computed(() => {
  switch (cintiaState.value) {
    case 'turning': return '/images/cintia-turning.svg'
    case 'facing': return '/images/cintia-facing.svg'
    default: return '/images/cintia-back.svg'
  }
})

function startGame() {
  gameState.value = 'playing'
  watchedTime.value = 0
  cintiaState.value = 'back'
  isHolding.value = false
  showWarning.value = false
  scheduleCintiaTurn()
}

function scheduleCintiaTurn() {
  const delay = (2 + Math.random() * 3) * 1000 // 2-5 seconds
  cintiaTimeout = setTimeout(() => {
    startCintiaTurn()
  }, delay)
}

function startCintiaTurn() {
  // Phase 1: turning (warning)
  cintiaState.value = 'turning'
  showWarning.value = true

  turningTimeout = setTimeout(() => {
    // Phase 2: facing
    cintiaState.value = 'facing'
    showWarning.value = false

    // Check if player is caught
    if (isHolding.value) {
      gameLost()
      return
    }

    // Stay facing for 0.5s then return
    facingTimeout = setTimeout(() => {
      cintiaState.value = 'back'
      scheduleCintiaTurn()
    }, 500)
  }, 1500)
}

function onPointerDown(e) {
  e.preventDefault()
  if (gameState.value !== 'playing') return
  if (cintiaState.value === 'facing') {
    gameLost()
    return
  }
  isHolding.value = true
  startWatchTimer()
}

function onPointerUp(e) {
  e.preventDefault()
  if (gameState.value !== 'playing') return
  isHolding.value = false
  stopWatchTimer()
}

function startWatchTimer() {
  stopWatchTimer()
  const tickRate = 50 // ms
  watchInterval = setInterval(() => {
    watchedTime.value += tickRate / 1000
    if (watchedTime.value >= TOTAL_TIME) {
      gameWon()
    }
  }, tickRate)
}

function stopWatchTimer() {
  if (watchInterval) {
    clearInterval(watchInterval)
    watchInterval = null
  }
}

function gameLost() {
  cleanup()
  emit('lose')
}

function gameWon() {
  cleanup()
  emit('win')
}

function cleanup() {
  stopWatchTimer()
  clearTimeout(cintiaTimeout)
  clearTimeout(turningTimeout)
  clearTimeout(facingTimeout)
  clearTimeout(returnTimeout)
  isHolding.value = false
}

onMounted(() => {
  // Prevent context menu on long press (mobile)
  document.addEventListener('contextmenu', preventDefault)
})

onUnmounted(() => {
  cleanup()
  document.removeEventListener('contextmenu', preventDefault)
})

function preventDefault(e) {
  e.preventDefault()
}
</script>

<template>
  <div class="level-one">
    <button class="menu-back-btn" @click="$emit('menu')">&larr; Menu</button>
    <!-- Intro -->
    <div v-if="gameState === 'intro'" class="level-intro">
      <div class="level-badge">Niveau 1</div>
      <h2>Regarde le vid&eacute;o de Cintia de Sa, sans te faire prendre par elle!</h2>
      <p class="instruction">Tiens appuy&eacute; sur le t&eacute;l&eacute;phone pour regarder.<br>Rel&acirc;che quand elle se retourne!</p>
      <button class="start-btn" @click="startGame">Regarder</button>
    </div>

    <!-- Playing -->
    <div v-else class="game-area">
      <!-- Cintia in background -->
      <div class="cintia-container" :class="cintiaState">
        <img :src="cintiaImage" alt="Cintia" class="cintia-sprite" />
      </div>

      <!-- Warning overlay -->
      <Transition name="warning">
        <div v-if="showWarning" class="warning-text">
          &#9888;&#65039; Attention! Cintia se retourne!
        </div>
      </Transition>

      <!-- Progress bar -->
      <div class="progress-container">
        <div class="progress-bar" :style="{ width: progress + '%' }"></div>
        <span class="progress-label">{{ watchedTime.toFixed(1) }}s / {{ TOTAL_TIME }}s</span>
      </div>

      <!-- Phone -->
      <div
        class="phone-area"
        @mousedown="onPointerDown"
        @mouseup="onPointerUp"
        @mouseleave="onPointerUp"
        @touchstart="onPointerDown"
        @touchend="onPointerUp"
        @touchcancel="onPointerUp"
      >
        <div class="phone" :class="{ raised: isHolding }">
          <div class="phone-notch"></div>
          <div class="phone-screen">
            <div class="video-placeholder" :class="{ playing: isHolding }">
              <div class="video-gradient"></div>
              <div class="video-content">
                <div class="video-icon">{{ isHolding ? '&#9654;&#65039;' : '&#9208;&#65039;' }}</div>
                <div class="video-title">Cintia de Sa</div>
                <div class="video-sub">@cinbrazil</div>
              </div>
              <div v-if="isHolding" class="video-bars">
                <div v-for="i in 5" :key="i" class="bar" :style="{ animationDelay: i * 0.1 + 's' }"></div>
              </div>
            </div>
          </div>
          <div class="phone-bar"></div>
        </div>
        <p class="hold-hint" :class="{ hidden: isHolding }">
          &#128071; Tiens appuy&eacute; pour regarder
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.level-one {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

/* ===== INTRO ===== */
.level-intro {
  text-align: center;
  max-width: 500px;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.2rem;
}

.level-badge {
  display: inline-block;
  background: linear-gradient(45deg, #FFD700, #FF6E40);
  color: #1a1a2e;
  font-weight: 700;
  font-size: 0.9rem;
  padding: 0.4rem 1.2rem;
  border-radius: 20px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.level-intro h2 {
  font-size: clamp(1.2rem, 4vw, 1.6rem);
  line-height: 1.4;
  color: white;
}

.instruction {
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.95rem;
  line-height: 1.5;
}

.start-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 1.2rem;
  font-weight: 700;
  padding: 0.9rem 2.5rem;
  border: none;
  border-radius: 50px;
  background: linear-gradient(45deg, #FF4081, #FF6E40);
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(255, 64, 129, 0.4);
  margin-top: 0.5rem;
}

.start-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 30px rgba(255, 64, 129, 0.6);
}

/* ===== GAME AREA ===== */
.game-area {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  padding: 1rem;
  position: relative;
}

/* ===== CINTIA ===== */
.cintia-container {
  position: absolute;
  top: 5%;
  left: 50%;
  transform: translateX(-50%);
  transition: transform 0.3s ease;
  z-index: 1;
}

.cintia-sprite {
  width: clamp(120px, 30vw, 180px);
  height: auto;
  transition: transform 0.3s ease;
}

.cintia-container.turning .cintia-sprite {
  animation: turnShake 0.3s ease;
}

.cintia-container.facing .cintia-sprite {
  animation: faceSnap 0.2s ease;
}

@keyframes turnShake {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-3deg); }
  75% { transform: rotate(3deg); }
}

@keyframes faceSnap {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

/* ===== WARNING ===== */
.warning-text {
  position: absolute;
  top: 45%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(255, 0, 0, 0.85);
  color: white;
  font-weight: 700;
  font-size: clamp(1rem, 3.5vw, 1.4rem);
  padding: 0.8rem 1.5rem;
  border-radius: 12px;
  z-index: 10;
  white-space: nowrap;
  animation: pulse 0.5s ease infinite alternate;
}

@keyframes pulse {
  from { transform: translate(-50%, -50%) scale(1); }
  to { transform: translate(-50%, -50%) scale(1.05); }
}

.warning-enter-active {
  transition: all 0.2s ease;
}
.warning-leave-active {
  transition: all 0.15s ease;
}
.warning-enter-from {
  opacity: 0;
  transform: translate(-50%, -50%) scale(0.8);
}
.warning-leave-to {
  opacity: 0;
}

/* ===== PROGRESS BAR ===== */
.progress-container {
  position: absolute;
  top: 1rem;
  left: 50%;
  transform: translateX(-50%);
  width: min(90%, 350px);
  height: 28px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 14px;
  overflow: hidden;
  z-index: 5;
  border: 2px solid rgba(255, 255, 255, 0.2);
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #76FF03, #00E676);
  border-radius: 14px;
  transition: width 0.05s linear;
}

.progress-label {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 0.8rem;
  font-weight: 700;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.5);
}

/* ===== PHONE ===== */
.phone-area {
  position: absolute;
  bottom: 2rem;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 5;
  cursor: pointer;
}

.phone {
  width: clamp(160px, 40vw, 220px);
  height: clamp(280px, 55vh, 400px);
  background: #1a1a2e;
  border-radius: 24px;
  border: 3px solid #444;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 8px;
  transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  transform: translateY(60%);
  box-shadow: 0 -5px 30px rgba(0, 0, 0, 0.5);
}

.phone.raised {
  transform: translateY(10%);
}

.phone-notch {
  width: 50%;
  height: 6px;
  background: #333;
  border-radius: 3px;
  margin-bottom: 6px;
}

.phone-screen {
  flex: 1;
  width: 100%;
  border-radius: 16px;
  overflow: hidden;
  background: #000;
}

.phone-bar {
  width: 35%;
  height: 4px;
  background: #555;
  border-radius: 2px;
  margin-top: 6px;
}

/* ===== VIDEO PLACEHOLDER ===== */
.video-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  background: #111;
}

.video-gradient {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    135deg,
    #FF4081 0%,
    #E040FB 25%,
    #7C4DFF 50%,
    #448AFF 75%,
    #18FFFF 100%
  );
  background-size: 400% 400%;
  opacity: 0.15;
  transition: opacity 0.3s;
}

.video-placeholder.playing .video-gradient {
  opacity: 0.6;
  animation: gradientMove 3s ease infinite;
}

@keyframes gradientMove {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.video-content {
  z-index: 2;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.video-icon {
  font-size: 2.5rem;
}

.video-title {
  font-size: 1.1rem;
  font-weight: 700;
}

.video-sub {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.6);
}

.video-bars {
  position: absolute;
  bottom: 20px;
  display: flex;
  gap: 4px;
  align-items: flex-end;
}

.bar {
  width: 6px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 3px;
  animation: barBounce 0.6s ease infinite alternate;
}

@keyframes barBounce {
  from { height: 8px; }
  to { height: 28px; }
}

.hold-hint {
  margin-top: 0.8rem;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
  transition: opacity 0.3s;
}

.hold-hint.hidden {
  opacity: 0;
}
</style>
