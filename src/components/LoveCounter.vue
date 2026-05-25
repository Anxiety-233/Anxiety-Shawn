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

const hoursTogether = computed(() => {
  const diff = now.value - startDate
  return Math.floor(diff / (1000 * 60 * 60))
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

const currentTime = computed(() => {
  const h = String(now.value.getHours()).padStart(2, '0')
  const m = String(now.value.getMinutes()).padStart(2, '0')
  const s = String(now.value.getSeconds()).padStart(2, '0')
  return `${h}:${m}:${s}`
})

onMounted(() => {
  gsap.from('.counter-card', { opacity: 0, y: 40, duration: 0.8, stagger: 0.15, ease: 'power3.out' })
  gsap.from('.time-display', { opacity: 0, scale: 0.9, duration: 1, delay: 0.5 })
})
</script>

<template>
  <div class="love-counter">
    <div class="glow-orb orb-1"></div>
    <div class="glow-orb orb-2"></div>

    <div class="header-section">
      <div class="heart-pulse">
        <span class="heart-icon">❤️</span>
        <span class="pulse-ring"></span>
        <span class="pulse-ring delay"></span>
      </div>
      <h2 class="section-title">恋爱时间计数器</h2>
      <p class="section-sub">记录我们在一起的每一刻</p>
    </div>

    <div class="time-display">
      <span class="time-text">{{ currentTime }}</span>
    </div>

    <div class="counter-cards">
      <div class="counter-card main-card">
        <div class="card-glow"></div>
        <div class="card-number">{{ daysTogether }}</div>
        <div class="card-label">天</div>
        <div class="card-sub">我们已经在一起</div>
      </div>
      <div class="counter-card">
        <div class="card-number small">{{ hoursTogether.toLocaleString() }}</div>
        <div class="card-label">小时</div>
        <div class="card-sub">每一小时都是甜的</div>
      </div>
      <div class="counter-card">
        <div class="card-number small">{{ nextAnniversary }}</div>
        <div class="card-label">天</div>
        <div class="card-sub">距离下一个纪念日</div>
      </div>
    </div>

    <p class="counter-footer">每一天，都是和你在一起的好日子 ✨</p>
  </div>
</template>

<style scoped>
.love-counter {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: calc(100vh - 76px);
  padding: 40px 20px;
  position: relative;
}

.glow-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.15;
  pointer-events: none;
}

.orb-1 {
  width: 300px;
  height: 300px;
  background: #ff6b9d;
  top: 10%;
  left: -10%;
  animation: drift 8s ease-in-out infinite;
}

.orb-2 {
  width: 250px;
  height: 250px;
  background: #764ba2;
  bottom: 10%;
  right: -10%;
  animation: drift 10s ease-in-out infinite reverse;
}

@keyframes drift {
  0%, 100% { transform: translate(0, 0); }
  50% { transform: translate(30px, -20px); }
}

.header-section {
  text-align: center;
  margin-bottom: 24px;
}

.heart-pulse {
  position: relative;
  display: inline-block;
  margin-bottom: 16px;
}

.heart-icon {
  font-size: 48px;
  display: block;
  animation: heartbeat 1.2s ease-in-out infinite;
}

@keyframes heartbeat {
  0%, 100% { transform: scale(1); }
  15% { transform: scale(1.15); }
  30% { transform: scale(1); }
  45% { transform: scale(1.1); }
}

.pulse-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 60px;
  height: 60px;
  margin: -30px 0 0 -30px;
  border-radius: 50%;
  border: 2px solid rgba(255, 107, 157, 0.4);
  animation: pulseRing 2s ease-out infinite;
}

.pulse-ring.delay {
  animation-delay: 1s;
}

@keyframes pulseRing {
  0% { transform: scale(0.8); opacity: 1; }
  100% { transform: scale(2); opacity: 0; }
}

.section-title {
  font-size: 1.6rem;
  font-weight: 600;
  margin-bottom: 8px;
  background: linear-gradient(135deg, #fff, #f0c4ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.section-sub {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.45);
}

.time-display {
  margin-bottom: 32px;
  padding: 8px 20px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
}

.time-text {
  font-size: 1.1rem;
  font-weight: 300;
  letter-spacing: 3px;
  color: rgba(255, 255, 255, 0.6);
  font-variant-numeric: tabular-nums;
}

.counter-cards {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  justify-content: center;
  max-width: 600px;
}

.counter-card {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 24px;
  padding: 28px 32px;
  text-align: center;
  backdrop-filter: blur(10px);
  min-width: 160px;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
}

.counter-card:hover {
  border-color: rgba(255, 107, 157, 0.25);
  transform: translateY(-4px);
  box-shadow: 0 8px 32px rgba(255, 107, 157, 0.1);
}

.counter-card.main-card {
  min-width: 200px;
  padding: 36px 40px;
  border-color: rgba(255, 107, 157, 0.15);
  background: linear-gradient(135deg, rgba(255, 107, 157, 0.06), rgba(196, 77, 255, 0.04));
}

.card-glow {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255, 107, 157, 0.08) 0%, transparent 60%);
  pointer-events: none;
}

.card-number {
  font-size: 3.2rem;
  font-weight: 700;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  line-height: 1.1;
  font-variant-numeric: tabular-nums;
}

.card-number.small {
  font-size: 2.2rem;
}

.card-label {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 4px;
  font-weight: 300;
}

.card-sub {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.35);
  margin-top: 8px;
}

.counter-footer {
  margin-top: 40px;
  color: rgba(255, 255, 255, 0.4);
  font-size: 0.85rem;
}
</style>
