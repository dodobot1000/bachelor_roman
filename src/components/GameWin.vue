<template>
  <div class="game-win">
    <div class="confetti" v-for="i in 30" :key="i" :style="confettiStyle(i)"></div>
    <div class="content">
      <div class="icon">&#127881;</div>
      <h1>BRAVO!</h1>
      <p>{{ message }}</p>
      <p class="sub">Roman serait fier de toi &#127870;</p>
      <button v-if="hasNext" class="next-btn active" @click="$emit('next')">
        Niveau suivant
      </button>
      <button v-else class="next-btn" disabled>
        Plus de niveaux (bient&ocirc;t...)
      </button>
      <button class="menu-back-btn" style="position:static;margin-top:0.5rem" @click="$emit('menu')">&larr; Menu</button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  message: { type: String, default: 'Bien jou\u00e9!' },
  hasNext: { type: Boolean, default: false },
})
defineEmits(['next', 'menu'])

const confettiStyle = (i) => ({
  left: `${Math.random() * 100}%`,
  animationDelay: `${Math.random() * 3}s`,
  animationDuration: `${2 + Math.random() * 3}s`,
  backgroundColor: ['#FFD700', '#FF4081', '#00E5FF', '#76FF03', '#E040FB'][i % 5],
  width: `${6 + Math.random() * 10}px`,
  height: `${6 + Math.random() * 10}px`,
})
</script>

<style scoped>
.game-win {
  text-align: center;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  background: radial-gradient(circle, rgba(255, 215, 0, 0.15) 0%, transparent 70%);
}

.content {
  z-index: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.icon {
  font-size: 5rem;
  animation: bounce 0.6s ease infinite alternate;
}

@keyframes bounce {
  from { transform: translateY(0); }
  to { transform: translateY(-15px); }
}

h1 {
  font-size: clamp(2rem, 6vw, 3.5rem);
  font-family: 'Pacifico', cursive;
  color: #FFD700;
}

p {
  font-size: 1.2rem;
  color: rgba(255, 255, 255, 0.9);
}

.sub {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.6);
}

.next-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 1.1rem;
  font-weight: 600;
  padding: 0.8rem 2.5rem;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50px;
  background: transparent;
  color: rgba(255, 255, 255, 0.4);
  cursor: not-allowed;
  margin-top: 1rem;
}

.next-btn.active {
  background: linear-gradient(45deg, #FF4081, #FF6E40);
  border: none;
  color: white;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(255, 64, 129, 0.4);
  transition: transform 0.2s, box-shadow 0.2s;
}

.next-btn.active:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 30px rgba(255, 64, 129, 0.6);
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
