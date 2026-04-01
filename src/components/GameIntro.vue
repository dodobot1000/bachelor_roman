<template>
  <div class="intro">
    <div class="confetti" v-for="i in 20" :key="i" :style="confettiStyle(i)"></div>
    <div class="content">
      <h1 class="title">Bachelor de Roman</h1>
      <p class="subtitle">Le jeu ultime du Bachelor!</p>
      <div class="decoration">
        <span>&#127870;</span>
        <span>&#127881;</span>
        <span>&#127870;</span>
      </div>
      <button class="play-btn" @click="$emit('play')">
        Jouer
      </button>
    </div>
  </div>
</template>

<script setup>
defineEmits(['play'])

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
  margin-bottom: 2rem;
}

.decoration {
  font-size: 2.5rem;
  margin-bottom: 2rem;
  display: flex;
  gap: 1rem;
  justify-content: center;
}

.play-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 1.4rem;
  font-weight: 700;
  padding: 1rem 3rem;
  border: none;
  border-radius: 50px;
  background: linear-gradient(45deg, #FF4081, #FF6E40);
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(255, 64, 129, 0.4);
}

.play-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 30px rgba(255, 64, 129, 0.6);
}

.play-btn:active {
  transform: scale(0.98);
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
