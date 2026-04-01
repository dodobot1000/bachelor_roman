<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['lose', 'win', 'menu'])

const gameState = ref('intro') // 'intro', 'loading', 'playing', 'reveal'

// Celebrity data: name, birthDate, wikipedia article title (pt.wikipedia)
const allCelebrities = [
  { name: 'Anitta', birth: '1993-03-30', wiki: 'Anitta' },
  { name: 'Manu Gavassi', birth: '1993-01-04', wiki: 'Manu_Gavassi' },
  { name: 'Camila Queiroz', birth: '1993-06-27', wiki: 'Camila_Queiroz' },
  { name: 'Giovanna Lancellotti', birth: '1993-05-21', wiki: 'Giovanna_Lancellotti' },
  { name: 'Laura Neiva', birth: '1993-09-21', wiki: 'Laura_Neiva' },
  { name: 'Bianca Andrade', birth: '1994-10-15', wiki: 'Bianca_Andrade' },
  { name: 'Isabelle Drummond', birth: '1994-04-12', wiki: 'Isabelle_Drummond' },
  { name: 'Bruna Marquezine', birth: '1995-08-04', wiki: 'Bruna_Marquezine' },
  { name: 'Marina Ruy Barbosa', birth: '1995-06-30', wiki: 'Marina_Ruy_Barbosa' },
  { name: 'Alice Wegmann', birth: '1995-11-03', wiki: 'Alice_Wegmann' },
  { name: 'Lexa', birth: '1995-02-22', wiki: 'Lexa_(cantora)' },
  { name: 'Ludmilla', birth: '1995-04-24', wiki: 'Ludmilla' },
  { name: 'Sasha Meneghel', birth: '1998-07-28', wiki: 'Sasha_Meneghel' },
  { name: 'Lu\u00edsa Sonza', birth: '1998-07-18', wiki: 'Lu%C3%ADsa_Sonza' },
  { name: 'Flavia Pavanelli', birth: '1998-03-02', wiki: 'Flavia_Pavanelli' },
  { name: 'Virginia Fonseca', birth: '1999-04-06', wiki: 'Virg%C3%ADnia_Fonseca' },
  { name: 'Giulia Be', birth: '1999-08-13', wiki: 'Giulia_Be' },
  { name: 'Rebeca Andrade', birth: '1999-05-08', wiki: 'Rebeca_Andrade' },
  { name: 'Klara Castanho', birth: '2000-10-06', wiki: 'Klara_Castanho' },
  { name: 'Larissa Manoela', birth: '2000-12-28', wiki: 'Larissa_Manoela' },
  { name: 'Jade Picon', birth: '2001-09-24', wiki: 'Jade_Picon' },
  { name: 'Giovanna Chaves', birth: '2001-12-04', wiki: 'Giovanna_Chaves' },
  { name: 'Maisa Silva', birth: '2002-05-22', wiki: 'Ma%C3%ADsa_Silva' },
  { name: 'Ana Castela', birth: '2003-11-16', wiki: 'Ana_Castela' },
  { name: 'Mel Maia', birth: '2004-05-03', wiki: 'Mel_Maia' },
  { name: 'Sophia Valverde', birth: '2005-08-30', wiki: 'Sophia_Valverde' },
  { name: 'MC Melody', birth: '2007-02-04', wiki: 'Melody_(cantora)' },
  { name: 'Rayssa Leal', birth: '2008-01-04', wiki: 'Rayssa_Leal' },
]

// Calculate age as of today
function getAge(birthStr) {
  const today = new Date()
  const birth = new Date(birthStr)
  let age = today.getFullYear() - birth.getFullYear()
  const m = today.getMonth() - birth.getMonth()
  if (m < 0 || (m === 0 && today.getDate() < birth.getDate())) {
    age--
  }
  return age
}

// Prepare celebrities with computed age
const celebrities = allCelebrities.map(c => ({
  ...c,
  age: getAge(c.birth),
  imageUrl: null,
  imageLoaded: false,
}))

// Image cache
const imageCache = reactive({})
const loadingImages = ref(true)

// Fetch image from Wikipedia API
async function fetchImage(celeb) {
  const wikis = ['pt', 'en']
  for (const lang of wikis) {
    try {
      const url = `https://${lang}.wikipedia.org/api/rest_v1/page/summary/${celeb.wiki}`
      const res = await fetch(url)
      if (!res.ok) continue
      const data = await res.json()
      // Use the thumbnail source directly — do not alter the URL
      if (data.thumbnail && data.thumbnail.source) {
        imageCache[celeb.name] = data.thumbnail.source
        return data.thumbnail.source
      }
    } catch {
      // try next wiki
    }
  }
  imageCache[celeb.name] = null
  return null
}

// Load all images
async function loadAllImages() {
  loadingImages.value = true
  const promises = celebrities.map(c => fetchImage(c))
  await Promise.allSettled(promises)
  loadingImages.value = false
}

// Game state
const rounds = ref([])
const currentRound = ref(0)
const score = ref(0)
const selectedAnswer = ref(null)
const showResult = ref(false)
const totalRounds = 8
let resultTimeout = null

// Generate rounds: pick 3 celebrities with different ages, ask about one
function generateRounds() {
  const available = celebrities.filter(c => imageCache[c.name])
  const generatedRounds = []

  for (let i = 0; i < totalRounds; i++) {
    // Shuffle and pick 3 with different ages
    const shuffled = [...available].sort(() => Math.random() - 0.5)
    const picked = []
    const usedAges = new Set()

    for (const c of shuffled) {
      if (!usedAges.has(c.age) && picked.length < 3) {
        picked.push(c)
        usedAges.add(c.age)
      }
    }

    if (picked.length < 3) {
      // Fallback: just pick 3 random
      picked.length = 0
      picked.push(...shuffled.slice(0, 3))
    }

    // The target is the youngest of the 3
    let targetIdx = 0
    for (let j = 1; j < picked.length; j++) {
      if (picked[j].age < picked[targetIdx].age) {
        targetIdx = j
      }
    }

    generatedRounds.push({
      choices: picked,
      targetIdx,
      targetAge: picked[targetIdx].age,
      targetName: picked[targetIdx].name,
    })
  }
  return generatedRounds
}

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

  const round = rounds.value[currentRound.value]
  if (idx === round.targetIdx) {
    score.value++
  }

  resultTimeout = setTimeout(() => {
    nextRound()
  }, 2500)
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

const currentRoundData = computed(() => rounds.value[currentRound.value])

onMounted(() => {})
onUnmounted(() => {
  clearTimeout(resultTimeout)
})
</script>

<template>
  <div class="level-four">
    <button class="menu-back-btn" @click="$emit('menu')">&larr; Menu</button>
    <!-- Intro -->
    <div v-if="gameState === 'intro'" class="level-intro">
      <div class="level-badge">Niveau 4</div>
      <h2>Laquelle est l&eacute;gale?</h2>
      <p class="instruction">
        On te montre 3 br&eacute;siliennes c&eacute;l&egrave;bres.<br />
        Devine laquelle a l'&acirc;ge demand&eacute;!<br />
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
    <div v-else-if="gameState === 'playing' && currentRoundData" class="game-area">
      <!-- Top bar -->
      <div class="top-bar">
        <div class="round-info">{{ currentRound + 1 }} / {{ totalRounds }}</div>
        <div class="score-info">Score: {{ score }}</div>
      </div>

      <!-- Question -->
      <div class="question">
        <span class="question-text">Laquelle a <strong>{{ currentRoundData.targetAge }} ans</strong> ?</span>
      </div>

      <!-- Photo cards -->
      <div class="cards">
        <div
          v-for="(celeb, idx) in currentRoundData.choices"
          :key="celeb.name + currentRound"
          class="card"
          :class="{
            correct: showResult && idx === currentRoundData.targetIdx,
            wrong: showResult && selectedAnswer === idx && idx !== currentRoundData.targetIdx,
            dimmed: showResult && idx !== currentRoundData.targetIdx && selectedAnswer !== idx,
          }"
          @click="selectAnswer(idx)"
        >
          <div class="card-img-wrapper">
            <img
              v-if="imageCache[celeb.name]"
              :src="imageCache[celeb.name]"
              :alt="celeb.name"
              class="card-img"
              loading="eager"
              referrerpolicy="no-referrer"
            />
            <div v-else class="card-placeholder">
              <span>&#128105;</span>
            </div>
          </div>
          <div class="card-name">{{ celeb.name }}</div>
          <Transition name="pop">
            <div v-if="showResult" class="card-age" :class="{ highlight: idx === currentRoundData.targetIdx }">
              {{ celeb.age }} ans
            </div>
          </Transition>
        </div>
      </div>

      <!-- Feedback -->
      <Transition name="pop">
        <div v-if="showResult" class="feedback" :class="{ correct: selectedAnswer === currentRoundData.targetIdx }">
          {{ selectedAnswer === currentRoundData.targetIdx
            ? 'Bonne r\u00e9ponse!'
            : 'Rat\u00e9! C\'\u00e9tait ' + currentRoundData.targetName }}
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
.level-four {
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
  background: linear-gradient(45deg, #E040FB, #FF4081);
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
  background: linear-gradient(45deg, #E040FB, #FF4081);
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(224, 64, 251, 0.4);
}

.start-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 30px rgba(224, 64, 251, 0.6);
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

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading p {
  color: rgba(255, 255, 255, 0.6);
}

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

.question-text {
  font-size: clamp(1rem, 3.5vw, 1.3rem);
}

.question-text strong {
  color: #EA80FC;
  font-size: 1.1em;
}

/* ===== CARDS ===== */
.cards {
  display: flex;
  gap: 0.8rem;
  width: min(95%, 500px);
  justify-content: center;
}

.card {
  flex: 1;
  max-width: 160px;
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
  transform: scale(1.05);
  box-shadow: 0 8px 30px rgba(0, 200, 83, 0.3);
}

.card.wrong {
  border-color: #FF1744;
  background: rgba(255, 23, 68, 0.15);
  animation: shakeCard 0.4s ease;
}

.card.dimmed {
  opacity: 0.4;
  cursor: default;
}

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
  font-size: clamp(0.7rem, 2.5vw, 0.9rem);
  font-weight: 600;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  width: 100%;
}

.card-age {
  padding: 0.3rem 0.5rem 0.5rem;
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.5);
}

.card-age.highlight {
  color: #69F0AE;
  font-weight: 700;
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
