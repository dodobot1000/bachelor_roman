<script setup>
import { ref, computed, onUnmounted } from 'vue'

const emit = defineEmits(['lose', 'win', 'menu'])

const gameState = ref('intro') // 'intro', 'playing'

const substances = [
  {
    id: 'cocaine',
    name: 'Coca\u00efne',
    image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/2/22/Peruvian_Flake_Cocaine.jpg/500px-Peruvian_Flake_Cocaine.jpg',
  },
  {
    id: 'sel',
    name: 'Sel',
    image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/a/a2/Salt_Crystals.JPG/500px-Salt_Crystals.JPG',
  },
  {
    id: 'ghb',
    name: 'GHB',
    image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/9/93/GHB_%28Gamma-Hydroxybutyric_Acid%29.jpg/500px-GHB_%28Gamma-Hydroxybutyric_Acid%29.jpg',
  },
  {
    id: 'sucre',
    name: 'Sucre',
    image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/5/56/Sugar_2xmacro.jpg/500px-Sugar_2xmacro.jpg',
  },
]

// 4 rounds, one per substance, shuffled order
function generateRounds() {
  return [...substances].sort(() => Math.random() - 0.5)
}

const rounds = ref([])
const currentRound = ref(0)
const score = ref(0)
const selectedAnswer = ref(null)
const showResult = ref(false)
let resultTimeout = null

const totalRounds = 4
const currentSubstance = computed(() => rounds.value[currentRound.value])
const isCorrect = computed(() => selectedAnswer.value === currentSubstance.value?.id)

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
  if (currentRound.value >= totalRounds - 1) {
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
    <button class="menu-back-btn" @click="$emit('menu')">&larr; Menu</button>
    <!-- Intro -->
    <div v-if="gameState === 'intro'" class="level-intro">
      <div class="level-badge">Niveau 2</div>
      <h2>Devine la substance!</h2>
      <p class="instruction">
        On te montre une substance suspecte.<br />
        &Agrave; toi de deviner ce que c'est!<br />
        <strong>3/4 bonnes r&eacute;ponses</strong> pour passer.
      </p>
      <button class="start-btn" @click="startGame">C'est parti!</button>
    </div>

    <!-- Playing -->
    <div v-else-if="currentSubstance" class="game-area">
      <!-- Score & Progress -->
      <div class="top-bar">
        <div class="round-info">{{ currentRound + 1 }} / {{ totalRounds }}</div>
        <div class="score-info">Score: {{ score }}</div>
      </div>

      <!-- Substance photo -->
      <div class="substance-card">
        <div class="substance-photo">
          <img
            :src="currentSubstance.image"
            :alt="'Substance ' + (currentRound + 1)"
            class="substance-img"
            referrerpolicy="no-referrer"
          />
        </div>
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
  gap: 1.2rem;
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

/* ===== SUBSTANCE PHOTO ===== */
.substance-card {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.substance-photo {
  width: clamp(240px, 60vw, 340px);
  aspect-ratio: 4 / 3;
  border-radius: 16px;
  overflow: hidden;
  border: 3px solid rgba(255, 255, 255, 0.15);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.5);
  background: #1a1a2e;
}

.substance-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
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
