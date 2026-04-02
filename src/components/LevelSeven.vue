<script setup>
import { ref, reactive, computed, onUnmounted } from 'vue'

const emit = defineEmits(['lose', 'win', 'menu'])

const gameState = ref('intro') // 'intro', 'loading', 'playing'

// People data: xx = cisgender women (XX chromosomes), other = trans women
const allPeople = [
  // Cisgender women (XX)
  { name: 'Anitta', wiki: 'Anitta', xx: true },
  { name: 'Bruna Marquezine', wiki: 'Bruna_Marquezine', xx: true },
  { name: 'Marina Ruy Barbosa', wiki: 'Marina_Ruy_Barbosa', xx: true },
  { name: 'Camila Queiroz', wiki: 'Camila_Queiroz', xx: true },
  { name: 'Jade Picon', wiki: 'Jade_Picon', xx: true },
  { name: 'Ludmilla', wiki: 'Ludmilla', xx: true },
  { name: 'Lu\u00edsa Sonza', wiki: 'Lu%C3%ADsa_Sonza', xx: true },
  { name: 'Larissa Manoela', wiki: 'Larissa_Manoela', xx: true },
  { name: 'Maisa Silva', wiki: 'Ma%C3%ADsa_Silva', xx: true },
  { name: 'Lexa', wiki: 'Lexa_(cantora)', xx: true },
  // Trans women (XY)
  { name: 'Pabllo Vittar', wiki: 'Pabllo_Vittar', xx: false },
  { name: 'Valentina Sampaio', wiki: 'Valentina_Sampaio', xx: false },
  { name: 'Roberta Close', wiki: 'Roberta_Close', xx: false },
  { name: 'Lea T', wiki: 'Lea_T', xx: false },
  { name: 'Linn da Quebrada', wiki: 'Linn_da_Quebrada', xx: false },
  { name: 'Ariadna Arantes', wiki: 'Ariadna_Arantes', xx: false },
]

const imageCache = reactive({})
const loadingImages = ref(true)

async function fetchImage(person) {
  const wikis = ['pt', 'en']
  for (const lang of wikis) {
    try {
      const url = `https://${lang}.wikipedia.org/api/rest_v1/page/summary/${person.wiki}`
      const res = await fetch(url)
      if (!res.ok) continue
      const data = await res.json()
      if (data.thumbnail && data.thumbnail.source) {
        imageCache[person.name] = data.thumbnail.source
        return
      }
    } catch { /* try next */ }
  }
  imageCache[person.name] = null
}

async function loadAllImages() {
  loadingImages.value = true
  await Promise.allSettled(allPeople.map(p => fetchImage(p)))
  loadingImages.value = false
}

// Game
const rounds = ref([])
const currentRound = ref(0)
const score = ref(0)
const selectedAnswer = ref(null)
const showResult = ref(false)
const totalRounds = 8
let resultTimeout = null

function generateRounds() {
  const available = allPeople.filter(p => imageCache[p.name])
  const xxPeople = available.filter(p => p.xx)
  const xyPeople = available.filter(p => !p.xx)
  const generatedRounds = []

  for (let i = 0; i < totalRounds; i++) {
    // Pick 2 people: 1 XX and 1 XY (or vice versa), randomly ordered
    const xx = xxPeople[Math.floor(Math.random() * xxPeople.length)]
    const xy = xyPeople[Math.floor(Math.random() * xyPeople.length)]
    const choices = Math.random() > 0.5 ? [xx, xy] : [xy, xx]
    const correctIdx = choices.findIndex(c => c.xx)

    generatedRounds.push({ choices, correctIdx })
  }
  return generatedRounds
}

const currentRoundData = computed(() => rounds.value[currentRound.value])

async function startGame() {
  gameState.value = 'loading'
  await loadAllImages()
  rounds.value = generateRounds()
  currentRound.value = 0
  score.value = 0
  selectedAnswer.value = null
  showResult.value = false
  gameState.value = 'playing'
}

function selectAnswer(idx) {
  if (showResult.value) return
  selectedAnswer.value = idx
  showResult.value = true
  if (idx === currentRoundData.value.correctIdx) {
    score.value++
  }
  resultTimeout = setTimeout(() => nextRound(), 2500)
}

function nextRound() {
  if (currentRound.value >= totalRounds - 1) {
    if (score.value >= 5) {
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

onUnmounted(() => { clearTimeout(resultTimeout) })
</script>

<template>
  <div class="level-seven">
    <button class="menu-back-btn" @click="$emit('menu')">&larr; Menu</button>

    <!-- Intro -->
    <div v-if="gameState === 'intro'" class="level-intro">
      <div class="level-badge">Niveau 7</div>
      <h2>Qui a 2 chromosomes X?</h2>
      <p class="instruction">
        On te montre 2 personnes.<br />
        Devine qui a les chromosomes XX!<br />
        <strong>5/{{ totalRounds }} bonnes r&eacute;ponses</strong> pour passer.
      </p>
      <button class="start-btn" @click="startGame">C'est parti!</button>
    </div>

    <!-- Loading -->
    <div v-else-if="gameState === 'loading'" class="loading">
      <div class="spinner"></div>
      <p>Chargement des photos...</p>
    </div>

    <!-- Playing -->
    <div v-else-if="currentRoundData" class="game-area">
      <div class="top-bar">
        <div class="round-info">{{ currentRound + 1 }} / {{ totalRounds }}</div>
        <div class="score-info">Score: {{ score }}</div>
      </div>

      <div class="question">
        <span class="question-text">Qui a <strong>2 chromosomes X</strong> ?</span>
      </div>

      <div class="cards">
        <div
          v-for="(person, idx) in currentRoundData.choices"
          :key="person.name + currentRound"
          class="card"
          :class="{
            correct: showResult && idx === currentRoundData.correctIdx,
            wrong: showResult && selectedAnswer === idx && idx !== currentRoundData.correctIdx,
            dimmed: showResult && idx !== currentRoundData.correctIdx && selectedAnswer !== idx,
          }"
          @click="selectAnswer(idx)"
        >
          <div class="card-img-wrapper">
            <img
              v-if="imageCache[person.name]"
              :src="imageCache[person.name]"
              :alt="person.name"
              class="card-img"
              referrerpolicy="no-referrer"
            />
            <div v-else class="card-placeholder">&#128105;</div>
          </div>
          <div class="card-name">{{ person.name }}</div>
          <Transition name="pop">
            <div v-if="showResult" class="card-label" :class="{ xx: person.xx }">
              {{ person.xx ? 'XX' : 'XY' }}
            </div>
          </Transition>
        </div>
      </div>

      <Transition name="pop">
        <div v-if="showResult" class="feedback" :class="{ correct: selectedAnswer === currentRoundData.correctIdx }">
          {{ selectedAnswer === currentRoundData.correctIdx ? 'Bonne r\u00e9ponse!' : 'Rat\u00e9!' }}
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
.level-seven {
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
  background: linear-gradient(45deg, #E040FB, #AA00FF);
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
  background: linear-gradient(45deg, #E040FB, #AA00FF);
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(224, 64, 251, 0.4);
}

.start-btn:hover {
  transform: scale(1.05);
}

/* ===== LOADING ===== */
.loading {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
}

.spinner {
  width: 48px;
  height: 48px;
  border: 4px solid rgba(255, 255, 255, 0.2);
  border-top-color: #E040FB;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

.loading p { color: rgba(255, 255, 255, 0.6); }

/* ===== GAME AREA ===== */
.game-area {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 1rem;
  gap: 1.2rem;
}

.top-bar {
  display: flex;
  justify-content: space-between;
  width: min(90%, 450px);
  font-size: 0.95rem;
  font-weight: 600;
}

.round-info {
  background: rgba(255, 255, 255, 0.1);
  padding: 0.3rem 1rem;
  border-radius: 20px;
}

.score-info {
  background: rgba(224, 64, 251, 0.3);
  padding: 0.3rem 1rem;
  border-radius: 20px;
  color: #EA80FC;
}

/* ===== QUESTION ===== */
.question {
  background: rgba(255, 255, 255, 0.08);
  padding: 0.8rem 1.5rem;
  border-radius: 14px;
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.question-text { font-size: clamp(1rem, 3.5vw, 1.3rem); }
.question-text strong { color: #EA80FC; }

/* ===== CARDS ===== */
.cards {
  display: flex;
  gap: 1rem;
  width: min(95%, 420px);
  justify-content: center;
}

.card {
  flex: 1;
  max-width: 200px;
  background: rgba(255, 255, 255, 0.06);
  border: 2px solid rgba(255, 255, 255, 0.15);
  border-radius: 16px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.25s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.card:hover:not(.correct):not(.wrong):not(.dimmed) {
  border-color: rgba(224, 64, 251, 0.5);
  transform: translateY(-4px);
  box-shadow: 0 8px 25px rgba(224, 64, 251, 0.2);
}

.card.correct {
  border-color: #00C853;
  background: rgba(0, 200, 83, 0.15);
  transform: scale(1.03);
}

.card.wrong {
  border-color: #FF1744;
  background: rgba(255, 23, 68, 0.15);
  animation: shakeCard 0.4s ease;
}

.card.dimmed { opacity: 0.4; cursor: default; }

@keyframes shakeCard {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

.card-img-wrapper {
  width: 100%;
  aspect-ratio: 3 / 4;
  overflow: hidden;
  background: #2a2a3e;
}

.card-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.card-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 3rem;
  background: #2a2a3e;
}

.card-name {
  padding: 0.5rem 0.3rem;
  font-size: clamp(0.75rem, 2.5vw, 0.95rem);
  font-weight: 600;
  text-align: center;
}

.card-label {
  padding: 0.3rem 0.8rem 0.5rem;
  font-size: 0.85rem;
  font-weight: 700;
  border-radius: 10px;
  margin-bottom: 0.4rem;
  background: rgba(255, 23, 68, 0.2);
  color: #FF8A80;
}

.card-label.xx {
  background: rgba(0, 200, 83, 0.2);
  color: #69F0AE;
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

.pop-enter-active { transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); }
.pop-leave-active { transition: all 0.2s ease; }
.pop-enter-from { opacity: 0; transform: scale(0.8); }
.pop-leave-to { opacity: 0; }
</style>
