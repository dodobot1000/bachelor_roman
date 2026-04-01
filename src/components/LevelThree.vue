<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['lose', 'win', 'menu'])

const gameState = ref('intro') // 'intro', 'playing'

// Game dimensions (responsive)
const gameWidth = ref(360)
const gameHeight = ref(600)
const gameRef = ref(null)

// Player
const player = reactive({ x: 50, y: 10, size: 28 }) // x/y in % of game area

// Gangsters
const gangsters = ref([])
let gangsterId = 0

// Buildings (decorative favela walls)
const buildings = ref([])

// Game progress
const progress = ref(0) // 0-100
const GAME_DURATION = 20 // seconds to survive
const speed = ref(1)

// Touch/drag
const isDragging = ref(false)
let dragStartX = 0
let dragStartPlayerX = 0

// Game loop
let animFrame = null
let lastTime = 0
let gangsterSpawnTimer = 0
let difficultyTimer = 0

function initGame() {
  player.x = 50
  player.y = 75
  gangsters.value = []
  buildings.value = []
  progress.value = 0
  speed.value = 1
  gangsterId = 0
  gangsterSpawnTimer = 0
  difficultyTimer = 0
  lastTime = 0

  // Generate initial buildings
  generateBuildings()
}

function generateBuildings() {
  buildings.value = []
  for (let i = 0; i < 12; i++) {
    const side = i % 2 === 0 ? 'left' : 'right'
    buildings.value.push({
      id: i,
      side,
      x: side === 'left' ? -2 + Math.random() * 5 : 78 + Math.random() * 5,
      y: i * 9 + Math.random() * 4,
      width: 18 + Math.random() * 8,
      height: 6 + Math.random() * 5,
      color: ['#8B6914', '#A0522D', '#CD853F', '#D2691E', '#B8860B'][Math.floor(Math.random() * 5)],
      windows: Math.floor(1 + Math.random() * 3),
    })
  }
}

function startGame() {
  initGame()
  gameState.value = 'playing'
  lastTime = performance.now()
  animFrame = requestAnimationFrame(gameLoop)
}

function gameLoop(timestamp) {
  if (gameState.value !== 'playing') return

  const dt = Math.min((timestamp - lastTime) / 1000, 0.05) // cap delta
  lastTime = timestamp

  // Update progress
  progress.value += (100 / GAME_DURATION) * dt

  if (progress.value >= 100) {
    gameWon()
    return
  }

  // Increase difficulty over time
  difficultyTimer += dt
  if (difficultyTimer > 4) {
    speed.value = Math.min(speed.value + 0.15, 2.5)
    difficultyTimer = 0
  }

  // Spawn gangsters
  gangsterSpawnTimer += dt
  const spawnRate = Math.max(0.8, 1.8 - progress.value * 0.01)
  if (gangsterSpawnTimer > spawnRate) {
    spawnGangster()
    gangsterSpawnTimer = 0
  }

  // Move gangsters
  updateGangsters(dt)

  // Scroll buildings
  updateBuildings(dt)

  // Check collisions
  if (checkCollisions()) {
    gameLost()
    return
  }

  animFrame = requestAnimationFrame(gameLoop)
}

function spawnGangster() {
  const fromSide = Math.random() > 0.5
  const g = {
    id: gangsterId++,
    x: fromSide
      ? (Math.random() > 0.5 ? -8 : 108)
      : 10 + Math.random() * 80,
    y: fromSide
      ? 10 + Math.random() * 70
      : -8,
    size: 24,
    speedX: 0,
    speedY: 0,
    type: Math.floor(Math.random() * 3), // visual variety
  }

  // Move toward player general area with some randomness
  if (fromSide) {
    g.speedX = g.x < 0 ? (15 + Math.random() * 20) : -(15 + Math.random() * 20)
    g.speedY = (Math.random() - 0.3) * 10
  } else {
    g.speedX = (Math.random() - 0.5) * 15
    g.speedY = 15 + Math.random() * 15
  }

  gangsters.value.push(g)
}

function updateGangsters(dt) {
  gangsters.value = gangsters.value.filter(g => {
    g.x += g.speedX * speed.value * dt
    g.y += g.speedY * speed.value * dt

    // Remove if off screen
    return g.x > -15 && g.x < 115 && g.y > -15 && g.y < 115
  })
}

function updateBuildings(dt) {
  buildings.value.forEach(b => {
    b.y += 8 * speed.value * dt
    if (b.y > 110) {
      b.y = -12
      b.height = 6 + Math.random() * 5
      b.color = ['#8B6914', '#A0522D', '#CD853F', '#D2691E', '#B8860B'][Math.floor(Math.random() * 5)]
      b.windows = Math.floor(1 + Math.random() * 3)
    }
  })
}

function checkCollisions() {
  const px = player.x
  const py = player.y
  const ps = 3.5 // collision radius in %

  for (const g of gangsters.value) {
    const dx = px - g.x
    const dy = py - g.y
    const dist = Math.sqrt(dx * dx + dy * dy)
    if (dist < ps + 3) {
      return true
    }
  }
  return false
}

function gameLost() {
  cancelAnimationFrame(animFrame)
  emit('lose')
}

function gameWon() {
  cancelAnimationFrame(animFrame)
  emit('win')
}

// ===== CONTROLS =====

// Keyboard
function onKeyDown(e) {
  if (gameState.value !== 'playing') return
  const moveSpeed = 3
  if (e.key === 'ArrowLeft' || e.key === 'a') {
    player.x = Math.max(5, player.x - moveSpeed)
  } else if (e.key === 'ArrowRight' || e.key === 'd') {
    player.x = Math.min(95, player.x + moveSpeed)
  } else if (e.key === 'ArrowUp' || e.key === 'w') {
    player.y = Math.max(5, player.y - moveSpeed)
  } else if (e.key === 'ArrowDown' || e.key === 's') {
    player.y = Math.min(95, player.y + moveSpeed)
  }
}

// Touch/Mouse
function onPointerDown(e) {
  if (gameState.value !== 'playing') return
  isDragging.value = true
  const pos = getPointerPos(e)
  dragStartX = pos.x
  dragStartPlayerX = player.x
  // Also handle tap-to-move
  movePlayerTo(pos)
}

function onPointerMove(e) {
  if (!isDragging.value || gameState.value !== 'playing') return
  e.preventDefault()
  const pos = getPointerPos(e)
  movePlayerTo(pos)
}

function onPointerUp() {
  isDragging.value = false
}

function getPointerPos(e) {
  const el = gameRef.value
  if (!el) return { x: 50, y: 50 }
  const rect = el.getBoundingClientRect()
  const clientX = e.touches ? e.touches[0].clientX : e.clientX
  const clientY = e.touches ? e.touches[0].clientY : e.clientY
  return {
    x: ((clientX - rect.left) / rect.width) * 100,
    y: ((clientY - rect.top) / rect.height) * 100,
  }
}

function movePlayerTo(pos) {
  // Smooth move toward pointer
  player.x = Math.max(5, Math.min(95, pos.x))
  player.y = Math.max(5, Math.min(95, pos.y))
}

onMounted(() => {
  window.addEventListener('keydown', onKeyDown)
})

onUnmounted(() => {
  cancelAnimationFrame(animFrame)
  window.removeEventListener('keydown', onKeyDown)
})

const progressPercent = computed(() => Math.min(100, progress.value))

function gangsterEmoji(type) {
  return ['&#128374;', '&#129399;', '&#128378;'][type] // various villain emojis
}
</script>

<template>
  <div class="level-three">
    <button class="menu-back-btn" @click="$emit('menu')">&larr; Menu</button>
    <!-- Intro -->
    <div v-if="gameState === 'intro'" class="level-intro">
      <div class="level-badge">Niveau 3</div>
      <h2>Fuir de la favela!</h2>
      <p class="instruction">
        Les gars sont en haut de la favela.<br />
        Descends sans te faire attraper par les gangsters!<br /><br />
        <strong>Clavier:</strong> fl&egrave;ches ou WASD<br />
        <strong>Mobile:</strong> touche et glisse pour bouger
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
      <!-- Sky gradient at top -->
      <div class="favela-bg"></div>

      <!-- Buildings (decoration) -->
      <div
        v-for="b in buildings"
        :key="b.id"
        class="building"
        :style="{
          left: b.x + '%',
          top: b.y + '%',
          width: b.width + '%',
          height: b.height + '%',
          background: b.color,
        }"
      >
        <div
          v-for="w in b.windows"
          :key="w"
          class="window"
          :style="{ left: (w * 22) + '%', top: '25%' }"
        ></div>
      </div>

      <!-- Road markings -->
      <div
        v-for="b in buildings"
        :key="'road-' + b.id"
        class="road-mark"
        :style="{ top: b.y + 3 + '%' }"
      ></div>

      <!-- Progress bar -->
      <div class="progress-container">
        <div class="progress-bar" :style="{ width: progressPercent + '%' }"></div>
        <span class="progress-label">
          {{ Math.floor(progressPercent) }}% descendu
        </span>
      </div>

      <!-- Player -->
      <div
        class="player"
        :style="{
          left: player.x + '%',
          top: player.y + '%',
        }"
      >
        <div class="player-body">&#127939;</div>
      </div>

      <!-- Gangsters -->
      <div
        v-for="g in gangsters"
        :key="g.id"
        class="gangster"
        :style="{
          left: g.x + '%',
          top: g.y + '%',
        }"
        v-html="gangsterEmoji(g.type)"
      >
      </div>

      <!-- Speed indicator -->
      <div class="speed-badge" v-if="speed > 1.3">
        &#9889; PLUS VITE!
      </div>
    </div>
  </div>
</template>

<style scoped>
.level-three {
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
  background: linear-gradient(45deg, #FF6E40, #FF3D00);
  color: white;
  font-weight: 700;
  font-size: 0.9rem;
  padding: 0.4rem 1.2rem;
  border-radius: 20px;
  text-transform: uppercase;
  letter-spacing: 1px;
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
  background: linear-gradient(45deg, #FF6E40, #FF3D00);
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(255, 110, 64, 0.4);
}

.start-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 30px rgba(255, 110, 64, 0.6);
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

.favela-bg {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    180deg,
    #2d1b0e 0%,
    #3e2723 20%,
    #4e342e 40%,
    #5d4037 60%,
    #6d4c41 80%,
    #795548 100%
  );
  z-index: 0;
}

/* ===== BUILDINGS ===== */
.building {
  position: absolute;
  z-index: 1;
  border-radius: 2px;
  border: 1px solid rgba(0, 0, 0, 0.3);
  box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.4);
}

.window {
  position: absolute;
  width: 30%;
  height: 40%;
  background: rgba(255, 200, 50, 0.4);
  border: 1px solid rgba(0, 0, 0, 0.3);
  border-radius: 1px;
}

.road-mark {
  position: absolute;
  left: 48%;
  width: 4%;
  height: 0.8%;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 2px;
  z-index: 1;
}

/* ===== PROGRESS ===== */
.progress-container {
  position: absolute;
  top: 0.8rem;
  left: 50%;
  transform: translateX(-50%);
  width: min(80%, 300px);
  height: 24px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 12px;
  overflow: hidden;
  z-index: 20;
  border: 2px solid rgba(255, 255, 255, 0.15);
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #FF6E40, #FF3D00);
  border-radius: 12px;
  transition: width 0.1s linear;
}

.progress-label {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 0.7rem;
  font-weight: 700;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.8);
}

/* ===== PLAYER ===== */
.player {
  position: absolute;
  z-index: 15;
  transform: translate(-50%, -50%);
  transition: left 0.08s linear, top 0.08s linear;
}

.player-body {
  font-size: 2rem;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.6));
  animation: playerBob 0.4s ease infinite alternate;
}

@keyframes playerBob {
  from { transform: translateY(0); }
  to { transform: translateY(-3px); }
}

/* ===== GANGSTERS ===== */
.gangster {
  position: absolute;
  z-index: 10;
  font-size: 1.8rem;
  transform: translate(-50%, -50%);
  filter: drop-shadow(0 2px 6px rgba(255, 0, 0, 0.4));
  transition: left 0.05s linear, top 0.05s linear;
}

/* ===== SPEED BADGE ===== */
.speed-badge {
  position: absolute;
  bottom: 1.5rem;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(255, 61, 0, 0.8);
  color: white;
  font-weight: 700;
  font-size: 0.85rem;
  padding: 0.4rem 1rem;
  border-radius: 20px;
  z-index: 20;
  animation: pulseBadge 0.5s ease infinite alternate;
}

@keyframes pulseBadge {
  from { transform: translateX(-50%) scale(1); }
  to { transform: translateX(-50%) scale(1.08); }
}
</style>
