<template>
  <div class="intro">
    <div class="confetti" v-for="i in 20" :key="i" :style="confettiStyle(i)"></div>
    <div class="content">
      <h1 class="title">Bachelor au Bresil</h1>
      <p class="subtitle">Le jeu ultime du Bachelor!</p>
      <div class="decoration">
        <span>&#127870;</span>
        <span>&#127881;</span>
        <span>&#127870;</span>
      </div>

      <!-- Menu -->
      <div v-if="!showLevels" class="menu">
        <button class="menu-btn primary" @click="$emit('play')">
          &#127918; Nouvelle partie
        </button>
        <button class="menu-btn secondary" @click="showLevels = true">
          &#128218; Tous les niveaux
        </button>
      </div>

      <!-- Level select -->
      <div v-else class="level-select">
        <button class="back-btn" @click="showLevels = false">
          &larr; Retour
        </button>
        <div class="levels-grid">
          <button
            v-for="level in levels"
            :key="level.id"
            class="level-card"
            @click="$emit('select-level', level.id)"
          >
            <span class="level-num">{{ level.id }}</span>
            <span class="level-name">{{ level.name }}</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

defineEmits(['play', 'select-level'])

const showLevels = ref(false)

const levels = [
  { id: 1, name: 'Regarde le vid\u00e9o' },
  { id: 2, name: 'Devine la substance' },
  { id: 3, name: 'Fuir de la favela' },
  { id: 4, name: 'Laquelle est l\u00e9gale?' },
  { id: 5, name: 'Quizz Br\u00e9sil' },
  { id: 6, name: 'Sors Yiz\u00e9 du bar' },
  { id: 7, name: 'Qui a 2 chromosomes X?' },
]

const confettiStyle = (i) => ({
  left: `${Math.random() * 100}%`,
  animationDelay: `${Math.random() * 3}s`,
  animationDuration: `${2 + Math.random() * 3}s`,
  backgroundColor: ['#FF4081', '#FFD700', '#00E5FF', '#76FF03', '#FF6E40'][i % 5],
  width: `${6 + Math.random() * 8}px`,
  height: `${6 + Math.random() * 8}px`,
})
</script>

<style scoped>
.intro {
  text-align: center;
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.content {
  z-index: 2;
  position: relative;
  width: min(90%, 500px);
}

.title {
  font-family: 'Pacifico', cursive;
  font-size: clamp(2.5rem, 8vw, 5rem);
  background: linear-gradient(45deg, #FF4081, #FFD700, #FF4081);
  background-size: 200% 200%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: shimmer 3s ease infinite;
  margin-bottom: 0.5rem;
  text-shadow: none;
}

@keyframes shimmer {
  0%, 100% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
}

.subtitle {
  font-size: clamp(1rem, 3vw, 1.5rem);
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 1.5rem;
}

.decoration {
  font-size: 2.5rem;
  margin-bottom: 2rem;
  display: flex;
  gap: 1rem;
  justify-content: center;
}

/* ===== MENU ===== */
.menu {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
  align-items: center;
}

.menu-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 1.15rem;
  font-weight: 700;
  padding: 0.9rem 2.5rem;
  border: none;
  border-radius: 50px;
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  width: 280px;
}

.menu-btn:hover {
  transform: scale(1.05);
}

.menu-btn:active {
  transform: scale(0.98);
}

.menu-btn.primary {
  background: linear-gradient(45deg, #FF4081, #FF6E40);
  box-shadow: 0 4px 20px rgba(255, 64, 129, 0.4);
}

.menu-btn.primary:hover {
  box-shadow: 0 6px 30px rgba(255, 64, 129, 0.6);
}

.menu-btn.secondary {
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.25);
  box-shadow: none;
}

.menu-btn.secondary:hover {
  background: rgba(255, 255, 255, 0.18);
  border-color: rgba(255, 255, 255, 0.4);
}

/* ===== LEVEL SELECT ===== */
.level-select {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
}

.back-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 0.9rem;
  font-weight: 600;
  padding: 0.4rem 1.2rem;
  border: none;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.7);
  cursor: pointer;
  transition: all 0.2s;
  align-self: flex-start;
}

.back-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  color: white;
}

.levels-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.8rem;
  width: 100%;
}

.level-card {
  font-family: 'Poppins', sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.4rem;
  padding: 1.2rem 0.8rem;
  border: 2px solid rgba(255, 255, 255, 0.15);
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.05);
  color: white;
  cursor: pointer;
  transition: all 0.2s;
}

.level-card:hover {
  border-color: rgba(255, 64, 129, 0.5);
  background: rgba(255, 64, 129, 0.1);
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(255, 64, 129, 0.2);
}

.level-card:active {
  transform: scale(0.97);
}

.level-num {
  font-size: 1.8rem;
  font-weight: 700;
  background: linear-gradient(45deg, #FF4081, #FFD700);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.level-name {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.6);
  text-align: center;
  line-height: 1.3;
}

.confetti {
  position: absolute;
  top: -20px;
  border-radius: 2px;
  animation: fall linear infinite;
  z-index: 1;
}

@keyframes fall {
  0% {
    transform: translateY(-10px) rotate(0deg);
    opacity: 1;
  }
  100% {
    transform: translateY(100vh) rotate(720deg);
    opacity: 0;
  }
}
</style>
