<script setup>
import { onMounted, ref } from 'vue'
import gsap from 'gsap'

const emit = defineEmits(['enter'])
const canvas = ref(null)
const showContent = ref(false)

onMounted(() => {
  showContent.value = true
  const cvs = canvas.value
  const ctx = cvs.getContext('2d')
  let w = cvs.width = window.innerWidth
  let h = cvs.height = window.innerHeight

  const stars = Array.from({ length: 200 }, () => ({
    x: Math.random() * w,
    y: Math.random() * h,
    r: Math.random() * 1.5 + 0.5,
    alpha: Math.random(),
    speed: Math.random() * 0.02 + 0.005
  }))

  const hearts = Array.from({ length: 15 }, () => ({
    x: Math.random() * w,
    y: Math.random() * h,
    size: Math.random() * 8 + 4,
    alpha: Math.random() * 0.4 + 0.1,
    speed: Math.random() * 0.3 + 0.1,
    drift: Math.random() * 0.5 - 0.25
  }))

  function drawHeart(x, y, size, alpha) {
    ctx.save()
    ctx.globalAlpha = alpha
    ctx.fillStyle = '#ff6b9d'
    ctx.beginPath()
    ctx.moveTo(x, y - size * 0.3)
    ctx.bezierCurveTo(x - size, y - size, x - size, y + size * 0.3, x, y + size)
    ctx.bezierCurveTo(x + size, y + size * 0.3, x + size, y - size, x, y - size * 0.3)
    ctx.fill()
    ctx.restore()
  }

  function animate() {
    ctx.clearRect(0, 0, w, h)

    stars.forEach(s => {
      s.alpha += s.speed
      if (s.alpha > 1 || s.alpha < 0) s.speed *= -1
      ctx.beginPath()
      ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(255, 255, 255, ${s.alpha})`
      ctx.fill()
    })

    hearts.forEach(p => {
      p.y -= p.speed
      p.x += p.drift
      if (p.y < -20) { p.y = h + 20; p.x = Math.random() * w }
      drawHeart(p.x, p.y, p.size, p.alpha)
    })

    requestAnimationFrame(animate)
  }
  animate()

  window.addEventListener('resize', () => {
    w = cvs.width = window.innerWidth
    h = cvs.height = window.innerHeight
  })

  gsap.from('.home-title', { opacity: 0, y: 30, duration: 1.2, delay: 0.3 })
  gsap.from('.home-subtitle', { opacity: 0, y: 20, duration: 1, delay: 0.8 })
  gsap.from('.enter-btn', { opacity: 0, scale: 0.8, duration: 0.8, delay: 1.3 })
})

function handleEnter() {
  gsap.to('.home-page', {
    opacity: 0,
    scale: 1.1,
    duration: 0.8,
    onComplete: () => emit('enter')
  })
}
</script>

<template>
  <div class="home-page">
    <canvas ref="canvas" class="bg-canvas"></canvas>
    <div class="home-content" v-if="showContent">
      <div class="planets">
        <div class="planet planet-1"></div>
        <div class="planet planet-2"></div>
      </div>
      <div class="moon"></div>
      <h1 class="home-title">我们的专属小宇宙</h1>
      <p class="home-subtitle">欢迎来到只属于我们的秘密空间</p>
      <button class="enter-btn" @click="handleEnter">
        <span>点击进入</span>
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M5 12h14M12 5l7 7-7 7"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<style scoped>
.home-page {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.bg-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.home-content {
  position: relative;
  z-index: 2;
  text-align: center;
}

.planets {
  position: absolute;
  top: -80px;
  left: 50%;
  transform: translateX(-50%);
  width: 200px;
  height: 80px;
}

.planet {
  position: absolute;
  border-radius: 50%;
  animation: float 4s ease-in-out infinite;
}

.planet-1 {
  width: 30px;
  height: 30px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  left: 40px;
  top: 10px;
  box-shadow: 0 0 20px rgba(102, 126, 234, 0.5);
}

.planet-2 {
  width: 22px;
  height: 22px;
  background: linear-gradient(135deg, #f093fb, #f5576c);
  right: 40px;
  top: 30px;
  animation-delay: -2s;
  box-shadow: 0 0 15px rgba(240, 147, 251, 0.5);
}

.moon {
  position: absolute;
  top: -120px;
  right: -60px;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffecd2, #fcb69f);
  box-shadow: 0 0 40px rgba(252, 182, 159, 0.4);
  animation: float 6s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.home-title {
  font-size: 2.2rem;
  font-weight: 700;
  margin-bottom: 16px;
  background: linear-gradient(135deg, #fff, #f0c4ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.home-subtitle {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 40px;
}

.enter-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 32px;
  border-radius: 30px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: #fff;
  font-size: 1rem;
  transition: all 0.3s;
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.4);
}

.enter-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 30px rgba(102, 126, 234, 0.6);
}
</style>
