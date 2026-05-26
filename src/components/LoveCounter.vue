<script setup>
import { ref, onMounted, computed, nextTick } from 'vue'
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

const timeParts = computed(() => {
  return {
    h: String(now.value.getHours()).padStart(2, '0'),
    m: String(now.value.getMinutes()).padStart(2, '0'),
    s: String(now.value.getSeconds()).padStart(2, '0')
  }
})

// ✨ 互动特效：点击爱心爆发粒子
const clickHearts = ref([])
let heartId = 0

function spawnHearts(e) {
  // 获取点击位置
  const rect = e.currentTarget.getBoundingClientRect()
  const originX = rect.width / 2
  const originY = rect.height / 2

  for (let i = 0; i < 12; i++) {
    const id = heartId++
    clickHearts.value.push({
      id,
      x: originX,
      y: originY,
      tx: (Math.random() - 0.5) * 200,
      ty: (Math.random() - 0.5) * 200 - 50,
      size: Math.random() * 15 + 10,
      rot: Math.random() * 360,
      delay: Math.random() * 0.1
    })

    // 动画结束后清理 DOM
    setTimeout(() => {
      clickHearts.value = clickHearts.value.filter(h => h.id !== id)
    }, 1500)
  }

  // 触发一次心跳放大的物理反馈
  gsap.fromTo(e.currentTarget,
      { scale: 0.8 },
      { scale: 1, duration: 0.5, ease: 'elastic.out(1.5, 0.5)' }
  )
}

// ✨ 互动特效：3D 卡片跟随鼠标倾斜
function handleCardMove(e) {
  const card = e.currentTarget
  const rect = card.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  const centerX = rect.width / 2
  const centerY = rect.height / 2

  // 计算倾斜角度 (最大倾斜 15 度)
  const rotateX = ((y - centerY) / centerY) * -15
  const rotateY = ((x - centerX) / centerX) * 15

  card.style.setProperty('--rx', `${rotateX}deg`)
  card.style.setProperty('--ry', `${rotateY}deg`)
  card.style.setProperty('--px', `${(x / rect.width) * 100}%`)
  card.style.setProperty('--py', `${(y / rect.height) * 100}%`)
}

function handleCardLeave(e) {
  const card = e.currentTarget
  // 鼠标离开时平滑回正
  card.style.setProperty('--rx', `0deg`)
  card.style.setProperty('--ry', `0deg`)
  card.style.setProperty('--px', `50%`)
  card.style.setProperty('--py', `50%`)
}

onMounted(() => {
  nextTick(() => {
    // 优化进场动画，更加丝滑浪漫
    const tl = gsap.timeline()
    tl.from('.heart-pulse', { scale: 0, opacity: 0, duration: 1, ease: 'elastic.out(1, 0.4)' })
        .from('.section-title', { y: 20, opacity: 0, duration: 0.8, ease: 'power3.out' }, '-=0.6')
        .from('.section-sub', { y: 20, opacity: 0, duration: 0.8, ease: 'power3.out' }, '-=0.6')
        .from('.time-display', { scale: 0.9, opacity: 0, duration: 0.8 }, '-=0.4')
        .from('.counter-card', { y: 50, opacity: 0, duration: 0.8, stagger: 0.15, ease: 'back.out(1.2)' }, '-=0.4')
        .from('.counter-footer', { opacity: 0, duration: 1 }, '-=0.2')
  })
})
</script>

<template>
  <div class="love-counter">
    <div class="glow-orb orb-1"></div>
    <div class="glow-orb orb-2"></div>
    <div class="stars-bg"></div>

    <div class="header-section">
      <div class="heart-pulse-container" @click="spawnHearts">
        <div class="heart-pulse">
          <span class="heart-icon">❤️</span>
          <span class="pulse-ring"></span>
          <span class="pulse-ring delay"></span>
        </div>

        <div
            v-for="heart in clickHearts"
            :key="heart.id"
            class="burst-heart"
            :style="{
            '--tx': `${heart.tx}px`,
            '--ty': `${heart.ty}px`,
            left: `${heart.x}px`,
            top: `${heart.y}px`,
            fontSize: `${heart.size}px`,
            transform: `rotate(${heart.rot}deg)`
          }"
        >❤️</div>
      </div>

      <h2 class="section-title">恋爱时间计数器</h2>
      <p class="section-sub">记录我们在一起的每一刻</p>
    </div>

    <div class="time-display">
      <div class="time-box">
        <span class="time-num">{{ timeParts.h }}</span>
        <span class="time-colon">:</span>
        <span class="time-num">{{ timeParts.m }}</span>
        <span class="time-colon">:</span>
        <span class="time-num">{{ timeParts.s }}</span>
      </div>
    </div>

    <div class="counter-cards">
      <div class="card-wrapper">
        <div class="counter-card main-card" @mousemove="handleCardMove" @mouseleave="handleCardLeave">
          <div class="card-glare"></div>
          <div class="card-content">
            <div class="card-number">{{ daysTogether }}</div>
            <div class="card-label">天</div>
            <div class="card-sub">我们已经在一起</div>
          </div>
        </div>
      </div>

      <div class="card-wrapper">
        <div class="counter-card" @mousemove="handleCardMove" @mouseleave="handleCardLeave">
          <div class="card-glare"></div>
          <div class="card-content">
            <div class="card-number small">{{ hoursTogether.toLocaleString() }}</div>
            <div class="card-label">小时</div>
            <div class="card-sub">每一小时都是甜的</div>
          </div>
        </div>
      </div>

      <div class="card-wrapper">
        <div class="counter-card" @mousemove="handleCardMove" @mouseleave="handleCardLeave">
          <div class="card-glare"></div>
          <div class="card-content">
            <div class="card-number small">{{ nextAnniversary }}</div>
            <div class="card-label">天</div>
            <div class="card-sub">距离下一个纪念日</div>
          </div>
        </div>
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
  padding: 40px 15px;
  position: relative;
  overflow: hidden;
  background: radial-gradient(circle at top, #1a1525, #0a0a10);
  font-family: 'PingFang SC', sans-serif;
}

/* --- 背景光晕与星空 --- */
.glow-orb { position: absolute; border-radius: 50%; filter: blur(90px); opacity: 0.15; pointer-events: none; }
.orb-1 { width: 350px; height: 350px; background: #ff6b9d; top: 5%; left: -10%; animation: drift 10s ease-in-out infinite alternate; }
.orb-2 { width: 300px; height: 300px; background: #764ba2; bottom: 5%; right: -10%; animation: drift 12s ease-in-out infinite alternate-reverse; }
@keyframes drift { 0% { transform: translate(0, 0) scale(1); } 100% { transform: translate(50px, 30px) scale(1.1); } }

.stars-bg {
  position: absolute; inset: 0; pointer-events: none;
  background-image: radial-gradient(rgba(255,255,255,0.1) 1px, transparent 1px);
  background-size: 40px 40px; opacity: 0.3; mask-image: radial-gradient(circle at center, black, transparent 80%);
}

/* --- 头部与互动爱心 --- */
.header-section { text-align: center; margin-bottom: 25px; z-index: 10; }

.heart-pulse-container { position: relative; display: inline-block; cursor: pointer; margin-bottom: 15px; padding: 20px; }
.heart-pulse { position: relative; z-index: 2; }
.heart-icon { font-size: 56px; display: block; filter: drop-shadow(0 0 15px rgba(255, 107, 157, 0.6)); animation: heartbeat 1.5s ease-in-out infinite; }
@keyframes heartbeat { 0%, 100% { transform: scale(1); } 15% { transform: scale(1.15); } 30% { transform: scale(1); } 45% { transform: scale(1.1); } }

.pulse-ring { position: absolute; top: 50%; left: 50%; width: 70px; height: 70px; margin: -35px 0 0 -35px; border-radius: 50%; border: 2px solid rgba(255, 107, 157, 0.5); animation: pulseRing 2s ease-out infinite; }
.pulse-ring.delay { animation-delay: 1s; }
@keyframes pulseRing { 0% { transform: scale(0.8); opacity: 1; } 100% { transform: scale(2.5); opacity: 0; } }

/* 爆炸粒子特效 */
.burst-heart {
  position: absolute; pointer-events: none; z-index: 1; opacity: 0;
  animation: burst 1s cubic-bezier(0.1, 0.8, 0.3, 1) forwards;
  filter: drop-shadow(0 0 5px rgba(255, 107, 157, 0.8));
}
@keyframes burst {
  0% { transform: translate(0, 0) scale(0) rotate(0deg); opacity: 1; }
  100% { transform: translate(var(--tx), var(--ty)) scale(1) rotate(180deg); opacity: 0; }
}

.section-title { font-size: 1.8rem; font-weight: 800; margin-bottom: 8px; background: linear-gradient(135deg, #fff, #f0c4ff); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.section-sub { font-size: 0.9rem; color: rgba(255, 255, 255, 0.5); }

/* --- 呼吸感时钟 --- */
.time-display { margin-bottom: 35px; position: relative; z-index: 10; }
.time-box {
  display: flex; align-items: center; justify-content: center; gap: 8px;
  padding: 12px 30px; border-radius: 16px; background: rgba(20, 15, 30, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.1); box-shadow: 0 10px 30px rgba(0,0,0,0.3), inset 0 0 15px rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);
}
.time-num { font-size: 1.6rem; font-weight: 700; color: #fff; text-shadow: 0 0 10px rgba(255, 255, 255, 0.3); font-variant-numeric: tabular-nums; width: 36px; text-align: center; }
.time-colon { font-size: 1.4rem; color: #ff6b9d; font-weight: 800; animation: blink 1s steps(2, start) infinite; text-shadow: 0 0 10px rgba(255, 107, 157, 0.5); margin: -4px; }
@keyframes blink { to { visibility: hidden; } }

/* --- 3D 悬浮数据卡片 --- */
.counter-cards { display: flex; gap: 20px; flex-wrap: wrap; justify-content: center; max-width: 800px; z-index: 10; perspective: 1000px; }

.card-wrapper { transform-style: preserve-3d; }

.counter-card {
  --rx: 0deg; --ry: 0deg; --px: 50%; --py: 50%;
  transform: rotateX(var(--rx)) rotateY(var(--ry));
  transition: transform 0.1s ease-out, box-shadow 0.3s;
  background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 24px; padding: 30px 20px; text-align: center; min-width: 170px;
  backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px);
  position: relative; overflow: hidden; transform-style: preserve-3d;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.2); cursor: default;
}

/* 镜面反光流光层 */
.card-glare {
  position: absolute; inset: 0; pointer-events: none; z-index: 1; mix-blend-mode: overlay;
  background: radial-gradient(circle at var(--px) var(--py), rgba(255, 255, 255, 0.4) 0%, transparent 60%);
  transition: background 0.1s;
}

.counter-card:hover { border-color: rgba(255, 107, 157, 0.3); box-shadow: 0 25px 50px rgba(0, 0, 0, 0.4), 0 0 20px rgba(255, 107, 157, 0.1); z-index: 10; }

.counter-card.main-card {
  min-width: 220px; padding: 40px 30px; border-color: rgba(255, 107, 157, 0.2);
  background: linear-gradient(135deg, rgba(255, 107, 157, 0.08), rgba(196, 77, 255, 0.05));
}

.card-content { transform: translateZ(30px); /* 让文字浮在卡片表面之上 */ position: relative; z-index: 2; pointer-events: none; }

.card-number {
  font-size: 3.5rem; font-weight: 800; line-height: 1.1; font-variant-numeric: tabular-nums;
  background: linear-gradient(135deg, #fff 0%, #ffb6c1 100%);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; filter: drop-shadow(0 4px 10px rgba(255, 107, 157, 0.3));
}
.card-number.small { font-size: 2.5rem; background: linear-gradient(135deg, #fff 0%, #d8b4fe 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.card-label { font-size: 0.9rem; color: rgba(255, 255, 255, 0.6); margin-top: 8px; font-weight: 600; letter-spacing: 2px; }
.card-sub { font-size: 0.75rem; color: rgba(255, 255, 255, 0.4); margin-top: 10px; }

.counter-footer { margin-top: 50px; color: rgba(255, 255, 255, 0.5); font-size: 0.9rem; z-index: 10; animation: pulseText 3s infinite alternate; }
@keyframes pulseText { 0% { opacity: 0.5; } 100% { opacity: 1; text-shadow: 0 0 10px rgba(255,255,255,0.3); } }
</style>