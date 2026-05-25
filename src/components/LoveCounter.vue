<script setup>
import { ref, onMounted, computed } from 'vue'
import gsap from 'gsap'

const startDate = new Date('2026-02-10')

const now = ref(new Date())
setInterval(() => { now.value = new Date() }, 1000)

const daysTogether = computed(() => {
  const diff = now.value - startDate
  return Math.floor(diff / (1000 * 60 * 60 * 24))
})

const nextAnniversary = computed(() => {
  const today = now.value
  let next = new Date(today.getFullYear(), startDate.getMonth(), startDate.getDate())
  if (next <= today) {
    next = new Date(today.getFullYear() + 1, startDate.getMonth(), startDate.getDate())
  }
  const diff = next - today
  return Math.floor(diff / (1000 * 60 * 60 * 24))
})

const heartScale = ref(1)

onMounted(() => {
  gsap.to(heartScale, {
    value: 1.2,
    duration: 0.6,
    repeat: -1,
    yoyo: true,
    ease: 'power1.inOut'
  })
  gsap.from('.counter-card', { opacity: 0, y: 40, duration: 1, stagger: 0.2 })
})
</script>

<template>
  <div class="love-counter">
    <div class="heart-bg" :style="{ transform: `scale(${heartScale})` }">❤️</div>
    <h2 class="section-title">恋爱时间计数器</h2>
    <div class="counter-cards">
      <div class="counter-card">
        <div class="card-number">{{ daysTogether }}</div>
        <div class="card-label">我们已经在一起</div>
        <div class="card-unit">天</div>
      </div>
      <div class="counter-card">
        <div class="card-number">{{ nextAnniversary }}</div>
        <div class="card-label">距离下一个纪念日</div>
        <div class="card-unit">天</div>
      </div>
    </div>
    <p class="counter-footer">每一天，都是和你在一起的好日子</p>
  </div>
</template>

<style scoped>
.love-counter {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: calc(100vh - 70px);
  padding: 40px 20px;
  position: relative;
}

.heart-bg {
  font-size: 60px;
  margin-bottom: 20px;
  filter: drop-shadow(0 0 20px rgba(255, 107, 157, 0.5));
}

.section-title {
  font-size: 1.5rem;
  margin-bottom: 40px;
  color: rgba(255, 255, 255, 0.9);
}

.counter-cards {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
  justify-content: center;
}

.counter-card {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  padding: 30px 40px;
  text-align: center;
  backdrop-filter: blur(10px);
  min-width: 180px;
}

.card-number {
  font-size: 3rem;
  font-weight: 700;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.card-label {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.6);
  margin-top: 8px;
}

.card-unit {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 4px;
}

.counter-footer {
  margin-top: 40px;
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.9rem;
}
</style>
