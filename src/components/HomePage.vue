<script setup>
import { onMounted, ref, onUnmounted } from 'vue'

const emit = defineEmits(['enter'])
const canvas = ref(null)

// 视差互动参数
const parallaxX = ref(0)
const parallaxY = ref(0)
let animationId = null

// 监听鼠标/手指滑动，计算视差偏移量
const handleMouseMove = (e) => {
  if (typeof window === 'undefined') return
  const clientX = e.touches ? e.touches[0].clientX : e.clientX
  const clientY = e.touches ? e.touches[0].clientY : e.clientY
  parallaxX.value = (clientX / window.innerWidth - 0.5) * 25
  parallaxY.value = (clientY / window.innerHeight - 0.5) * 25
}

onMounted(() => {
  const cvs = canvas.value
  if (!cvs) return
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
    ctx.globalAlpha = Math.max(0, Math.min(1, alpha)) // 安全限制，防止白屏
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
      ctx.fillStyle = `rgba(255, 255, 255, ${Math.max(0, Math.min(1, s.alpha))})`
      ctx.fill()
    })

    hearts.forEach(p => {
      p.y -= p.speed
      p.x += p.drift
      if (p.y < -20) { p.y = h + 20; p.x = Math.random() * w }
      drawHeart(p.x, p.y, p.size, p.alpha)
    })

    animationId = requestAnimationFrame(animate)
  }
  animate()

  window.addEventListener('resize', () => {
    if (cvs) {
      w = cvs.width = window.innerWidth
      h = cvs.height = window.innerHeight
    }
  })
})

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId)
})

function handleEnter() {
  // 没有任何延迟，点击瞬间立刻跳转
  emit('enter')
}
</script>

<template>
  <div
      class="home-page"
      @mousemove="handleMouseMove"
      @touchmove.passive="handleMouseMove"
  >
    <canvas ref="canvas" class="bg-canvas"></canvas>

    <div class="home-content">

      <div class="parallax-layer" :style="{ transform: `translate(${parallaxX}px, ${parallaxY}px)` }">

        <div class="glass-card">
          <div class="planets-group">
            <div class="moon"></div>
            <div class="planet planet-1"></div>
            <div class="planet planet-2"></div>
          </div>

          <h1 class="home-title">我们的专属小宇宙</h1>
          <p class="home-subtitle">欢迎来到只属于我们的秘密空间</p>

          <button class="enter-btn" @click="handleEnter">
            <span>点击进入</span>
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="M5 12h14M12 5l7 7-7 7"/>
            </svg>
          </button>
        </div>

      </div>
    </div>
  </div>
</template>

<style scoped>
.home-page {
  position: relative;
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  /* 深邃质感的星空背景 */
  background: linear-gradient(135deg, #0b071a 0%, #1a1025 50%, #05020a 100%);
  font-family: 'PingFang SC', sans-serif;
}

.bg-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  pointer-events: none;
}

.home-content {
  position: relative;
  z-index: 2;
  width: 100%;
  padding: 20px;
  display: flex;
  justify-content: center;
}

/* --- 视差跟随层 --- */
.parallax-layer {
  transition: transform 0.1s ease-out; /* 让鼠标跟随显得平滑 */
  width: 100%;
  max-width: 480px;
}

/* --- 毛玻璃主卡片 --- */
.glass-card {
  position: relative;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  border-left: 1px solid rgba(255, 255, 255, 0.2);
  padding: 60px 40px 50px;
  border-radius: 28px;
  text-align: center;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.4);
  /* 原生 CSS 进场动画：绝对稳定 */
  animation: cardShow 1s cubic-bezier(0.175, 0.885, 0.32, 1.275) backwards;
  animation-delay: 0.2s;
}

/* --- 天体装饰层 --- */
.planets-group {
  position: absolute;
  top: -60px;
  left: 0;
  width: 100%;
  height: 100px;
  pointer-events: none;
}

.planet, .moon {
  position: absolute;
  border-radius: 50%;
  animation: float 6s ease-in-out infinite;
}

.moon {
  top: -30px;
  right: -20px;
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, #ffecd2, #fcb69f);
  box-shadow: 0 0 35px rgba(252, 182, 159, 0.5);
}

.planet-1 {
  width: 45px;
  height: 45px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  left: -15px;
  top: 20px;
  box-shadow: 0 0 25px rgba(102, 126, 234, 0.6);
  animation-delay: -1.5s;
}

.planet-2 {
  width: 20px;
  height: 20px;
  background: linear-gradient(135deg, #f093fb, #f5576c);
  right: 60px;
  top: 70px;
  animation-delay: -3s;
  box-shadow: 0 0 15px rgba(240, 147, 251, 0.6);
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}

/* --- 文本美化 --- */
.home-title {
  font-size: 2.2rem;
  font-weight: 800;
  margin: 0 0 12px 0;
  background: linear-gradient(135deg, #ffffff, #f0c4ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 2px 8px rgba(0,0,0,0.4));
  /* 标题进场动画 */
  animation: fadeUp 0.8s ease-out backwards;
  animation-delay: 0.4s;
}

.home-subtitle {
  font-size: 1.05rem;
  color: rgba(255, 255, 255, 0.75);
  margin: 0 0 40px 0;
  letter-spacing: 1px;
  /* 副标题进场动画 */
  animation: fadeUp 0.8s ease-out backwards;
  animation-delay: 0.6s;
}

/* --- 绝不再消失的按钮 --- */
.enter-btn {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 15px 42px;
  border-radius: 40px;
  background: linear-gradient(135deg, #764ba2, #667eea);
  color: #fff;
  font-size: 1.1rem;
  font-weight: 600;
  border: 1px solid rgba(255, 255, 255, 0.2);
  cursor: pointer;
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  /* 原生 CSS 进场动画：稳如泰山 */
  animation: buttonPop 0.8s cubic-bezier(0.175, 0.885, 0.32, 1.275) backwards;
  animation-delay: 0.8s;
}

/* 纯 CSS 处理的按钮悬浮反馈 */
.enter-btn:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 35px rgba(102, 126, 234, 0.6);
  background: linear-gradient(135deg, #875fc2, #7890f0);
}

.enter-btn:active {
  transform: translateY(2px) scale(0.95);
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.enter-btn svg {
  transition: transform 0.3s ease;
}

.enter-btn:hover svg {
  transform: translateX(4px);
}

/* --- 稳定可靠的动画定义 --- */
@keyframes cardShow {
  0% { opacity: 0; transform: translateY(40px) scale(0.95); }
  100% { opacity: 1; transform: translateY(0) scale(1); }
}

@keyframes fadeUp {
  0% { opacity: 0; transform: translateY(20px); }
  100% { opacity: 1; transform: translateY(0); }
}

@keyframes buttonPop {
  0% { opacity: 0; transform: translateY(20px) scale(0.8); }
  100% { opacity: 1; transform: translateY(0) scale(1); }
}

/* 移动端适配 */
@media (max-width: 768px) {
  .glass-card { padding: 50px 25px 40px; }
  .home-title { font-size: 1.8rem; }
  .moon { width: 60px; height: 60px; right: -10px; top: -20px; }
  .planet-1 { width: 35px; height: 35px; left: -10px; }
}
</style>