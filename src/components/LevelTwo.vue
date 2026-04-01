<script setup>
import { ref, computed, onUnmounted } from 'vue'

const emit = defineEmits(['lose', 'win'])

const gameState = ref('intro') // 'intro', 'playing', 'reveal', 'result'

const substances = [
  {
    id: 'cocaine',
    name: 'Coca\u00efne',
    color: '#f5f5f0',
    texture: 'fine',
    description: 'Poudre blanche fine et brillante',
    hint: 'Souvent en lignes...',
  },
  {
    id: 'sel',
    name: 'Sel',
    color: '#ffffff',
    texture: 'crystal',
    description: 'Cristaux blancs translucides',
    hint: 'On en met dans la soupe!',
  },
  {
    id: 'ghb',
    name: 'GHB',
    color: '#e8e8f0',
    texture: 'liquid',
    description: 'Liquide transparent inodore',
    hint: 'Aussi appel\u00e9 "drogue du viol"',
  },
  {
    id: 'sucre',
    name: 'Sucre',
    color: '#fffef5',
    texture: 'granular',
    description: 'Granul\u00e9s blancs fins',
    hint: 'Dans ton caf\u00e9 le matin!',
  },
]

// Generate rounds - each substance appears, shuffled
function generateRounds() {
  const rounds = []
  // 5 rounds with random substances
  for (let i = 0; i < 5; i++) {
    rounds.push(substances[Math.floor(Math.random() * substances.length)])
  }
  return rounds
}

const rounds = ref(generateRounds())
const currentRound = ref(0)
const score = ref(0)
const selectedAnswer = ref(null)
const showResult = ref(false)
let resultTimeout = null

const currentSubstance = computed(() => rounds.value[currentRound.value])
const isCorrect = computed(() => selectedAnswer.value === currentSubstance.value.id)
const totalRounds = computed(() => rounds.value.length)

// Shuffle choices for each round
const shuffledChoices = computed(() => {
  return [...substances].sort(() => Math.random() - 0.5)
})

function startGame() {
  rounds.value = generateRounds()
  currentRound.value = 0
  score.value = 0
  selectedAnswer.value = null
  showResult.value = false
  gameState.value = 'playing'
}

function selectAnswer(id) {
  if (showResult.value) return
  selectedAnswer.value = id
  showResult.value = true

  if (id === currentSubstance.value.id) {
    score.value++
  }

  resultTimeout = setTimeout(() => {
    nextRound()
  }, 2000)
}

function nextRound() {
  if (currentRound.value >= totalRounds.value - 1) {
    // Game finished
    if (score.value >= 3) {
      emit('win')
    } else {
      emit('lose')
    }
    return
  }
  currentRound.value++
  selectedAnswer.value = null
  showResult.value = false
}

onUnmounted(() => {
  clearTimeout(resultTimeout)
})
</script>

<template>
  <div class="level-two">
    <!-- Intro -->
    <div v-if="gameState === 'intro'" class="level-intro">
      <div class="level-badge">Niveau 2</div>
      <h2>Devine la substance!</h2>
      <p class="instruction">
        On te montre une substance suspecte.<br />
        &Agrave; toi de deviner ce que c'est!<br />
        <strong>3/5 bonnes r&eacute;ponses</strong> pour passer.
      </p>
      <button class="start-btn" @click="startGame">C'est parti!</button>
    </div>

    <!-- Playing -->
    <div v-else class="game-area">
      <!-- Score & Progress -->
      <div class="top-bar">
        <div class="round-info">{{ currentRound + 1 }} / {{ totalRounds }}</div>
        <div class="score-info">Score: {{ score }}</div>
      </div>

      <!-- Substance display -->
      <div class="substance-card">
        <div class="substance-visual" :class="currentSubstance.texture">
          <div class="substance-inner">
            <!-- Fine powder -->
            <template v-if="currentSubstance.texture === 'fine'">
              <div class="powder-pile">
                <div v-for="i in 40" :key="i" class="powder-dot"
                  :style="{
                    left: 20 + Math.random() * 60 + '%',
                    top: 25 + Math.random() * 50 + '%',
                    width: 2 + Math.random() * 4 + 'px',
                    height: 2 + Math.random() * 4 + 'px',
                    opacity: 0.5 + Math.random() * 0.5,
                    background: currentSubstance.color,
                  }"
                ></div>
                <div class="powder-line" v-for="i in 3" :key="'l'+i"
                  :style="{
                    top: 30 + i * 15 + '%',
                    left: '25%',
                    width: '50%',
                  }"
                ></div>
              </div>
            </template>

            <!-- Crystal -->
            <template v-if="currentSubstance.texture === 'crystal'">
              <div class="crystal-pile">
                <div v-for="i in 25" :key="i" class="crystal"
                  :style="{
                    left: 15 + Math.random() * 70 + '%',
                    top: 20 + Math.random() * 60 + '%',
                    width: 3 + Math.random() * 8 + 'px',
                    height: 3 + Math.random() * 8 + 'px',
                    transform: 'rotate(' + Math.random() * 360 + 'deg)',
                    opacity: 0.6 + Math.random() * 0.4,
                  }"
                ></div>
              </div>
            </template>

            <!-- Liquid -->
            <template v-if="currentSubstance.texture === 'liquid'">
              <div class="liquid-container">
                <div class="glass">
                  <div class="liquid-fill"></div>
                  <div class="liquid-shine"></div>
                </div>
              </div>
            </template>

            <!-- Granular -->
            <template v-if="currentSubstance.texture === 'granular'">
              <div class="granular-pile">
                <div v-for="i in 35" :key="i" class="granule"
                  :style="{
                    left: 15 + Math.random() * 70 + '%',
                    top: 20 + Math.random() * 60 + '%',
                    width: 2 + Math.random() * 5 + 'px',
                    height: 2 + Math.random() * 5 + 'px',
                    opacity: 0.4 + Math.random() * 0.6,
                  }"
                ></div>
              </div>
            </template>
          </div>
        </div>
        <p class="substance-hint">{{ currentSubstance.hint }}</p>
      </div>

      <!-- Choices -->
      <div class="choices">
        <button
          v-for="choice in shuffledChoices"
          :key="choice.id"
          class="choice-btn"
          :class="{
            correct: showResult && choice.id === currentSubstance.id,
            wrong: showResult && selectedAnswer === choice.id && choice.id !== currentSubstance.id,
            disabled: showResult,
          }"
          @click="selectAnswer(choice.id)"
          :disabled="showResult"
        >
          {{ choice.name }}
        </button>
      </div>

      <!-- Result feedback -->
      <Transition name="pop">
        <div v-if="showResult" class="feedback" :class="{ correct: isCorrect }">
          {{ isCorrect ? 'Bonne r\u00e9ponse!' : 'Mauvaise r\u00e9ponse! C\'\u00e9tait: ' + currentSubstance.name }}
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
.level-two {
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
  background: linear-gradient(45deg, #00E5FF, #7C4DFF);
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
  background: linear-gradient(45deg, #00E5FF, #7C4DFF);
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(0, 229, 255, 0.3);
}

.start-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 30px rgba(0, 229, 255, 0.5);
}

/* ===== GAME AREA ===== */
.game-area {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 1.5rem;
  gap: 1.5rem;
}

.top-bar {
  display: flex;
  justify-content: space-between;
  width: min(90%, 400px);
  font-size: 1rem;
  font-weight: 600;
}

.round-info {
  background: rgba(255, 255, 255, 0.1);
  padding: 0.4rem 1rem;
  border-radius: 20px;
}

.score-info {
  background: rgba(124, 77, 255, 0.3);
  padding: 0.4rem 1rem;
  border-radius: 20px;
  color: #E0B0FF;
}

/* ===== SUBSTANCE CARD ===== */
.substance-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.substance-visual {
  width: clamp(200px, 50vw, 280px);
  height: clamp(150px, 35vw, 200px);
  background: #2a2a3e;
  border-radius: 16px;
  border: 2px solid rgba(255, 255, 255, 0.1);
  overflow: hidden;
  position: relative;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.4);
}

.substance-inner {
  width: 100%;
  height: 100%;
  position: relative;
}

.substance-hint {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
  font-style: italic;
}

/* Powder dots */
.powder-dot {
  position: absolute;
  border-radius: 50%;
}

.powder-line {
  position: absolute;
  height: 3px;
  background: rgba(245, 245, 240, 0.6);
  border-radius: 2px;
}

/* Crystal */
.crystal {
  position: absolute;
  background: rgba(255, 255, 255, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.3);
  clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);
}

/* Liquid */
.liquid-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.glass {
  width: 60px;
  height: 100px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 0 0 8px 8px;
  border-top: none;
  position: relative;
  overflow: hidden;
  clip-path: polygon(5% 0%, 95% 0%, 85% 100%, 15% 100%);
}

.liquid-fill {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 70%;
  background: rgba(200, 200, 240, 0.2);
  animation: liquidWave 2s ease infinite;
}

.liquid-shine {
  position: absolute;
  top: 30%;
  left: 20%;
  width: 8px;
  height: 30px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 4px;
  transform: rotate(-15deg);
}

@keyframes liquidWave {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-3px); }
}

/* Granular */
.granule {
  position: absolute;
  background: rgba(255, 254, 245, 0.7);
  border-radius: 1px;
}

/* ===== CHOICES ===== */
.choices {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.8rem;
  width: min(90%, 400px);
}

.choice-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 1rem;
  font-weight: 600;
  padding: 1rem;
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.05);
  color: white;
  cursor: pointer;
  transition: all 0.2s;
}

.choice-btn:hover:not(.disabled) {
  background: rgba(255, 255, 255, 0.15);
  border-color: rgba(255, 255, 255, 0.4);
  transform: scale(1.03);
}

.choice-btn:active:not(.disabled) {
  transform: scale(0.97);
}

.choice-btn.correct {
  background: rgba(0, 200, 83, 0.3);
  border-color: #00C853;
  color: #69F0AE;
}

.choice-btn.wrong {
  background: rgba(255, 23, 68, 0.3);
  border-color: #FF1744;
  color: #FF8A80;
  animation: shakeBtn 0.4s ease;
}

.choice-btn.disabled {
  cursor: default;
  opacity: 0.6;
}

@keyframes shakeBtn {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

/* ===== FEEDBACK ===== */
.feedback {
  font-size: 1.1rem;
  font-weight: 700;
  padding: 0.6rem 1.5rem;
  border-radius: 12px;
  background: rgba(255, 23, 68, 0.2);
  color: #FF8A80;
}

.feedback.correct {
  background: rgba(0, 200, 83, 0.2);
  color: #69F0AE;
}

.pop-enter-active {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.pop-leave-active {
  transition: all 0.2s ease;
}
.pop-enter-from {
  opacity: 0;
  transform: scale(0.8);
}
.pop-leave-to {
  opacity: 0;
}
</style>
