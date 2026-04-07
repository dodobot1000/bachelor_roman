<template>
  <div class="game-over">
    <div class="content">
      <div class="icon">&#128561;</div>
      <h1>Tu as &eacute;chou&eacute;</h1>
      <img :src="failImg" alt="Fail" class="cintia-img" />
      <button class="retry-btn" @click="$emit('retry')">
        Recommencer
      </button>
      <button class="menu-back-btn" style="position:static;margin-top:0.5rem" @click="$emit('menu')">&larr; Menu</button>
    </div>
  </div>
</template>

<script setup>
defineEmits(['retry', 'menu'])
const failImg = `${import.meta.env.BASE_URL}images/fail.svg`
</script>

<style scoped>
.game-over {
  text-align: center;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle, rgba(200, 0, 0, 0.3) 0%, transparent 70%);
}

.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.icon {
  font-size: 4rem;
  animation: shake 0.5s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20% { transform: translateX(-10px); }
  40% { transform: translateX(10px); }
  60% { transform: translateX(-10px); }
  80% { transform: translateX(10px); }
}

h1 {
  font-size: clamp(1.5rem, 5vw, 2.5rem);
  color: #FF4081;
}

p {
  font-size: 1.1rem;
  color: rgba(255, 255, 255, 0.8);
}

.cintia-img {
  width: 150px;
  height: auto;
  margin: 1rem 0;
}

.retry-btn {
  font-family: 'Poppins', sans-serif;
  font-size: 1.2rem;
  font-weight: 600;
  padding: 0.8rem 2.5rem;
  border: none;
  border-radius: 50px;
  background: linear-gradient(45deg, #FF4081, #FF6E40);
  color: white;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 20px rgba(255, 64, 129, 0.4);
}

.retry-btn:hover {
  transform: scale(1.05);
}
</style>
