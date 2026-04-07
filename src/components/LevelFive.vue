<script setup>
import { ref, computed, onUnmounted } from 'vue'

const emit = defineEmits(['lose', 'win', 'menu'])

const gameState = ref('intro')

// Pool of 15 questions, 5 will be picked randomly each game
const allQuestions = [
  {
    question: 'Quelle est la capitale du Br\u00e9sil?',
    choices: ['Rio de Janeiro', 'S\u00e3o Paulo', 'Bras\u00edlia', 'Salvador'],
    answer: 2,
  },
  {
    question: 'Quel est le plat national br\u00e9silien?',
    choices: ['Tacos', 'Feijoada', 'Paella', 'Ceviche'],
    answer: 1,
  },
  {
    question: 'Combien d\'\u00e9toiles figurent sur le drapeau du Br\u00e9sil?',
    choices: ['13', '22', '27', '50'],
    answer: 2,
  },
  {
    question: 'Quel fleuve traverse la for\u00eat amazonienne?',
    choices: ['Le Nil', 'Le Danube', 'L\'Amazone', 'Le Mississippi'],
    answer: 2,
  },
  {
    question: 'Combien de fois le Br\u00e9sil a gagn\u00e9 la Coupe du Monde de football?',
    choices: ['3', '4', '5', '6'],
    answer: 2,
  },
  {
    question: 'Quelle danse est originaire du Br\u00e9sil?',
    choices: ['Tango', 'Salsa', 'Samba', 'Bachata'],
    answer: 2,
  },
  {
    question: 'Dans quelle ville se d\u00e9roule le plus grand carnaval du monde?',
    choices: ['Venise', 'La Nouvelle-Orl\u00e9ans', 'Rio de Janeiro', 'Trinidad'],
    answer: 2,
  },
  {
    question: 'Quelle est la langue officielle du Br\u00e9sil?',
    choices: ['Espagnol', 'Portugais', 'Fran\u00e7ais', 'Anglais'],
    answer: 1,
  },
  {
    question: 'Quel est le nom de la c\u00e9l\u00e8bre statue \u00e0 Rio de Janeiro?',
    choices: ['Statue de la Libert\u00e9', 'Christ R\u00e9dempteur', 'Le Penseur', 'David'],
    answer: 1,
  },
  {
    question: 'Quelle boisson alcoolis\u00e9e est typiquement br\u00e9silienne?',
    choices: ['Tequila', 'Pisco', 'Cacha\u00e7a', 'Rhum'],
    answer: 2,
  },
  {
    question: 'Quel cocktail br\u00e9silien est fait avec de la cacha\u00e7a et du citron vert?',
    choices: ['Mojito', 'Caipirinha', 'Daiquiri', 'Margarita'],
    answer: 1,
  },
  {
    question: 'Quel art martial br\u00e9silien m\u00e9lange danse et combat?',
    choices: ['Jiu-Jitsu', 'Muay Thai', 'Capoeira', 'Krav Maga'],
    answer: 2,
  },
  {
    question: 'Quel est le plus grand stade du Br\u00e9sil?',
    choices: ['Maracan\u00e3', 'Morumbi', 'Al\u00e9ncar', 'Arena Corinthians'],
    answer: 0,
  },
  {
    question: 'Quelle ville br\u00e9silienne est la plus peupl\u00e9e?',
    choices: ['Rio de Janeiro', 'Bras\u00edlia', 'Salvador', 'S\u00e3o Paulo'],
    answer: 3,
  },
  {
    question: 'Quel joueur br\u00e9silien est surnomm\u00e9 "O Rei" (Le Roi)?',
    choices: ['Ronaldinho', 'Neymar', 'Pel\u00e9', 'Ronaldo'],
    answer: 2,
  },
]

function pickQuestions() {
  const shuffled = [...allQuestions].sort(() => Math.random() - 0.5)
  return shuffled.slice(0, 5)
}

const questions = ref([])
const currentQ = ref(0)
const score = ref(0)
const selectedAnswer = ref(null)
const showResult = ref(false)
let resultTimeout = null

const currentQuestion = computed(() => questions.value[currentQ.value])
const isCorrect = computed(() => selectedAnswer.value === currentQuestion.value?.answer)

function startGame() {
  questions.value = pickQuestions()
  currentQ.value = 0
  score.value = 0
  selectedAnswer.value = null
  showResult.value = false
  gameState.value = 'playing'
}

function selectAnswer(idx) {
  if (showResult.value) return
  selectedAnswer.value = idx
  showResult.value = true

  if (idx === currentQuestion.value.answer) {
    score.value++
  }

  resultTimeout = setTimeout(() => {
    nextQuestion()
  }, 2000)
}

function nextQuestion() {
  if (currentQ.value >= 4) {
    if (score.value >= 3) {
      emit('win')
    } else {
      emit('lose')
    }
    return
  }
  currentQ.value++
  selectedAnswer.value = null
  showResult.value = false
}

onUnmounted(() => {
  clearTimeout(resultTimeout)
})
</script>

<template>
  <div class="level-five">
    <button class="menu-back-btn" @click="$emit('menu')">&larr; Menu</button>

    <!-- Intro -->
    <div v-if="gameState === 'intro'" class="level-intro">
      <div class="level-badge">Niveau 1</div>
      <h2>Quizz Br&eacute;sil!</h2>
      <p class="instruction">
        5 questions de connaissances g&eacute;n&eacute;rales sur le Br&eacute;sil.<br />
        <strong>3/5 bonnes r&eacute;ponses</strong> pour passer.
      </p>
      <button class="start-btn" @click="startGame">C'est parti!</button>
    </div>

    <!-- Playing -->
    <div v-else-if="currentQuestion" class="game-area">
      <!-- Top bar -->
      <div class="top-bar">
        <div class="round-info">{{ currentQ + 1 }} / 5</div>
        <div class="score-info">Score: {{ score }}</div>
      </div>

      <!-- Question -->
      <div class="question-card">
        <div class="question-icon">&#127463;&#127479;</div>
        <p class="question-text">{{ currentQuestion.question }}</p>
      </div>

      <!-- Choices -->
      <div class="choices">
        <button
          v-for="(choice, idx) in currentQuestion.choices"
          :key="idx"
          class="choice-btn"
          :class="{
            correct: showResult && idx === currentQuestion.answer,
            wrong: showResult && selectedAnswer === idx && idx !== currentQuestion.answer,
            disabled: showResult,
          }"
          @click="selectAnswer(idx)"
          :disabled="showResult"
        >
          {{ choice }}
        </button>
      </div>

      <!-- Feedback -->
      <Transition name="pop">
        <div v-if="showResult" class="feedback" :class="{ correct: isCorrect }">
          {{ isCorrect ? 'Bonne r\u00e9ponse!' : 'Rat\u00e9! C\'\u00e9tait: ' + currentQuestion.choices[currentQuestion.answer] }}
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
.level-five {
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
  background: linear-gradient(45deg, #00C853, #FFD600);
  color: #1a1a2e;
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
  background: linear-gradient(45deg, #00C853, #FFD600);
  color: #1a1a2e;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(0, 200, 83, 0.3);
}

.start-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 30px rgba(0, 200, 83, 0.5);
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
  width: min(90%, 420px);
  font-size: 0.95rem;
  font-weight: 600;
}

.round-info {
  background: rgba(255, 255, 255, 0.1);
  padding: 0.3rem 1rem;
  border-radius: 20px;
}

.score-info {
  background: rgba(0, 200, 83, 0.25);
  padding: 0.3rem 1rem;
  border-radius: 20px;
  color: #69F0AE;
}

/* ===== QUESTION ===== */
.question-card {
  background: rgba(255, 255, 255, 0.06);
  border: 2px solid rgba(255, 255, 255, 0.12);
  border-radius: 18px;
  padding: 1.5rem;
  width: min(90%, 420px);
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.8rem;
}

.question-icon {
  font-size: 2.2rem;
}

.question-text {
  font-size: clamp(1rem, 3.5vw, 1.2rem);
  line-height: 1.5;
  color: white;
}

/* ===== CHOICES ===== */
.choices {
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
  width: min(90%, 420px);
}

.choice-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 1rem;
  font-weight: 600;
  padding: 0.9rem 1.2rem;
  border: 2px solid rgba(255, 255, 255, 0.15);
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.05);
  color: white;
  cursor: pointer;
  transition: all 0.2s;
  text-align: left;
}

.choice-btn:hover:not(.disabled) {
  background: rgba(255, 255, 255, 0.12);
  border-color: rgba(255, 255, 255, 0.35);
  transform: translateX(4px);
}

.choice-btn:active:not(.disabled) {
  transform: scale(0.98);
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
  font-size: 1.05rem;
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
