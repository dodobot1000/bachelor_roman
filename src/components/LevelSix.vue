<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['lose', 'win', 'menu'])

const gameState = ref('intro') // 'intro', 'playing'
const gameRef = ref(null)

const baseUrl = import.meta.env.BASE_URL
const yizeImg = `${baseUrl}images/yize.jpg`

// Player position (%)
const player = reactive({ x: 50, y: 80 })

// Yizé position (chases player)
const yize = reactive({ x: 50, y: 30, speedX: 0, speedY: 0 })

// Obstacles (tables, chairs)
const obstacles = ref([])

// Timer
const timeLeft = ref(15)
const GAME_DURATION = 15

// Animation
let animFrame = null
let lastTime = 0

// Touch
const isDragging = ref(false)

function generateObstacles() {
  obstacles.value = [
    { id: 1, x: 15, y: 25, w: 14, h: 8, type: 'table' },
    { id: 2, x: 60, y: 20, w: 12, h: 8, type: 'table' },
    { id: 3, x: 35, y: 50, w: 14, h: 8, type: 'table' },
    { id: 4, x: 75, y: 55, w: 12, h: 8, type: 'table' },
    { id: 5, x: 10, y: 65, w: 10, h: 6, type: 'chair' },
    { id: 6, x: 50, y: 75, w: 10, h: 6, type: 'chair' },
  ]
}

function startGame() {
  player.x = 50
  player.y = 85
  yize.x = 50
  yize.y = 20
  timeLeft.value = GAME_DURATION
  generateObstacles()
  gameState.value = 'playing'
  lastTime = performance.now()
  animFrame = requestAnimationFrame(gameLoop)
}

function gameLoop(timestamp) {
  if (gameState.value !== 'playing') return
  const dt = Math.min((timestamp - lastTime) / 1000, 0.05)
  lastTime = timestamp

  // Countdown
  timeLeft.value -= dt
  if (timeLeft.value <= 0) {
    timeLeft.value = 0
    gameWon()
    return
  }

  // Yizé chases player with increasing speed
  const elapsed = GAME_DURATION - timeLeft.value
  const chaseSpeed = 12 + elapsed * 1.5 // gets faster over time
  const dx = player.x - yize.x
  const dy = player.y - yize.y
  const dist = Math.sqrt(dx * dx + dy * dy)
  if (dist > 0) {
    yize.x += (dx / dist) * chaseSpeed * dt
    yize.y += (dy / dist) * chaseSpeed * dt
  }

  // Add some wobble to make Yizé feel drunk/aggressive
  yize.x += Math.sin(timestamp / 200) * 0.3
  yize.y += Math.cos(timestamp / 300) * 0.2

  // Check collision
  const collisionDist = Math.sqrt(
    (player.x - yize.x) ** 2 + (player.y - yize.y) ** 2
  )
  if (collisionDist < 6) {
    gameLost()
    return
  }

  animFrame = requestAnimationFrame(gameLoop)
}

function gameLost() {
  cancelAnimationFrame(animFrame)
  emit('lose')
}

function gameWon() {
  cancelAnimationFrame(animFrame)
  emit('win')
}

// Controls
function onKeyDown(e) {
  if (gameState.value !== 'playing') return
  const speed = 3.5
  if (e.key === 'ArrowLeft' || e.key === 'a') player.x = Math.max(3, player.x - speed)
  else if (e.key === 'ArrowRight' || e.key === 'd') player.x = Math.min(97, player.x + speed)
  else if (e.key === 'ArrowUp' || e.key === 'w') player.y = Math.max(3, player.y - speed)
  else if (e.key === 'ArrowDown' || e.key === 's') player.y = Math.min(97, player.y + speed)
}

function onPointerDown(e) {
  if (gameState.value !== 'playing') return
  isDragging.value = true
  movePlayerTo(e)
}
function onPointerMove(e) {
  if (!isDragging.value) return
  e.preventDefault()
  movePlayerTo(e)
}
function onPointerUp() { isDragging.value = false }

function movePlayerTo(e) {
  const el = gameRef.value
  if (!el) return
  const rect = el.getBoundingClientRect()
  const cx = e.touches ? e.touches[0].clientX : e.clientX
  const cy = e.touches ? e.touches[0].clientY : e.clientY
  player.x = Math.max(3, Math.min(97, ((cx - rect.left) / rect.width) * 100))
  player.y = Math.max(3, Math.min(97, ((cy - rect.top) / rect.height) * 100))
}

onMounted(() => { window.addEventListener('keydown', onKeyDown) })
onUnmounted(() => {
  cancelAnimationFrame(animFrame)
  window.removeEventListener('keydown', onKeyDown)
})
</script>

<template>
  <div class="level-six">
    <button class="menu-back-btn" @click="$emit('menu')">&larr; Menu</button>

    <!-- Intro -->
    <div v-if="gameState === 'intro'" class="level-intro">
      <div class="level-badge">Niveau 6</div>
      <div class="yize-preview">
        <img :src="yizeImg" alt="Yiz\u00e9" class="yize-preview-img" />
      </div>
      <h2>Sors Yiz&eacute; du bar!</h2>
      <p class="instruction">
        Yiz&eacute; veut se battre!<br />
        Sors du bar en s&eacute;curit&eacute;.<br />
        Survis <strong>15 secondes</strong> sans te faire attraper!<br /><br />
        <strong>Clavier:</strong> fl&egrave;ches ou WASD<br />
        <strong>Mobile:</strong> touche et glisse
      </p>
      <button class="start-btn" @click="startGame">Courir!</button>
    </div>

    <!-- Playing -->
    <div
      v-else
      ref="gameRef"
      class="game-area"
      @mousedown="onPointerDown"
      @mousemove="onPointerMove"
      @mouseup="onPointerUp"
      @mouseleave="onPointerUp"
      @touchstart.prevent="onPointerDown"
      @touchmove.prevent="onPointerMove"
      @touchend="onPointerUp"
      @touchcancel="onPointerUp"
    >
      <!-- Bar background -->
      <div class="bar-bg"></div>

      <!-- Bar counter -->
      <div class="bar-counter"></div>
      <div class="bar-counter-label">BAR</div>

      <!-- Obstacles (tables) -->
      <div
        v-for="obs in obstacles"
        :key="obs.id"
        class="obstacle"
        :class="obs.type"
        :style="{ left: obs.x + '%', top: obs.y + '%', width: obs.w + '%', height: obs.h + '%' }"
      ></div>

      <!-- Timer -->
      <div class="timer" :class="{ danger: timeLeft < 5 }">
        {{ timeLeft.toFixed(1) }}s
      </div>

      <!-- Player -->
      <div class="player" :style="{ left: player.x + '%', top: player.y + '%' }">
        &#127939;
      </div>

      <!-- Yizé -->
      <div class="yize" :style="{ left: yize.x + '%', top: yize.y + '%' }">
        <img :src="yizeImg" alt="Yiz\u00e9" class="yize-face" />
        <div class="yize-rage">&#129324;</div>
      </div>

      <!-- Exit door -->
      <div class="exit-sign">&#128682; SORTIE</div>
    </div>
  </div>
</template>

<style scoped>
.level-six {
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
  gap: 1rem;
}

.level-badge {
  display: inline-block;
  background: linear-gradient(45deg, #FF1744, #D50000);
  color: white;
  font-weight: 700;
  font-size: 0.9rem;
  padding: 0.4rem 1.2rem;
  border-radius: 20px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.yize-preview {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  overflow: hidden;
  border: 4px solid #FF1744;
  box-shadow: 0 0 20px rgba(255, 23, 68, 0.5);
}

.yize-preview-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.level-intro h2 {
  font-size: clamp(1.4rem, 5vw, 2rem);
  color: white;
}

.instruction {
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.95rem;
  line-height: 1.6;
}

.start-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 1.2rem;
  font-weight: 700;
  padding: 0.9rem 2.5rem;
  border: none;
  border-radius: 50px;
  background: linear-gradient(45deg, #FF1744, #D50000);
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(255, 23, 68, 0.4);
}

.start-btn:hover {
  transform: scale(1.05);
}

/* ===== GAME AREA ===== */
.game-area {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
  cursor: crosshair;
  touch-action: none;
}

.bar-bg {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, #1a0a00 0%, #2d1810 30%, #3e2015 60%, #1a0a00 100%);
  z-index: 0;
}

/* Bar counter at top */
.bar-counter {
  position: absolute;
  top: 0;
  left: 10%;
  width: 80%;
  height: 8%;
  background: linear-gradient(180deg, #5d3a1a, #8B4513);
  border-radius: 0 0 8px 8px;
  z-index: 2;
  border: 2px solid #3e2010;
  border-top: none;
}

.bar-counter-label {
  position: absolute;
  top: 1.5%;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0.7rem;
  font-weight: 700;
  color: rgba(255, 200, 100, 0.6);
  z-index: 3;
  letter-spacing: 3px;
}

/* Obstacles */
.obstacle {
  position: absolute;
  z-index: 2;
  border-radius: 6px;
}

.obstacle.table {
  background: #5d3a1a;
  border: 2px solid #3e2010;
  box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.5);
}

.obstacle.chair {
  background: #4a2a10;
  border: 1px solid #3e2010;
  border-radius: 50%;
}

/* Timer */
.timer {
  position: absolute;
  top: 0.8rem;
  right: 0.8rem;
  font-size: 1.4rem;
  font-weight: 700;
  background: rgba(0, 0, 0, 0.6);
  padding: 0.4rem 1rem;
  border-radius: 12px;
  z-index: 20;
  color: white;
}

.timer.danger {
  color: #FF1744;
  animation: timerPulse 0.5s ease infinite alternate;
}

@keyframes timerPulse {
  from { transform: scale(1); }
  to { transform: scale(1.1); }
}

/* Player */
.player {
  position: absolute;
  z-index: 15;
  font-size: 2rem;
  transform: translate(-50%, -50%);
  transition: left 0.08s linear, top 0.08s linear;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.6));
}

/* Yizé */
.yize {
  position: absolute;
  z-index: 14;
  transform: translate(-50%, -50%);
  transition: left 0.05s linear, top 0.05s linear;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.yize-face {
  width: clamp(45px, 10vw, 60px);
  height: clamp(45px, 10vw, 60px);
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid #FF1744;
  box-shadow: 0 0 15px rgba(255, 23, 68, 0.6);
  animation: yizeShake 0.15s ease infinite alternate;
}

@keyframes yizeShake {
  from { transform: rotate(-5deg); }
  to { transform: rotate(5deg); }
}

.yize-rage {
  font-size: 1rem;
  margin-top: -4px;
}

/* Exit sign */
.exit-sign {
  position: absolute;
  bottom: 0.5rem;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0.8rem;
  font-weight: 700;
  color: #69F0AE;
  background: rgba(0, 0, 0, 0.5);
  padding: 0.3rem 0.8rem;
  border-radius: 8px;
  z-index: 5;
  letter-spacing: 1px;
}
</style>
