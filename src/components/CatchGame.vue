<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const canvasRef = ref(null)
const gameState = ref('ready')
const score = ref(0)
const lives = ref(5)
const combo = ref(0)
const level = ref(1)
const bestScore = ref(parseInt(localStorage.getItem('catchBest') || '0'))

let ctx, w, h, animId
let paddle = { x: 0, width: 70, height: 12 }
let items = []
let particles = []
let floatingTexts = []
let stars = []
let frameCount = 0
let mouseX = 0
let canvasRect = null // ✨ 修复2：缓存画布边界，避免鼠标移动时浏览器疯狂重排

// 状态与特效变量
let shakeAmount = 0
let activeBuffs = { magnet: 0, slow: 0, shield: false }

const itemTypes = [
  { emoji: '💖', type: 'score', score: 10, color: '#ff6b9d', weight: 40 },
  { emoji: '💕', type: 'score', score: 20, color: '#ec407a', weight: 25 },
  { emoji: '⭐', type: 'score', score: 50, color: '#ffd700', weight: 12 },
  { emoji: '💎', type: 'score', score: 100, color: '#64c8ff', weight: 5 },
  { emoji: '💔', type: 'bomb', score: -30, color: '#a0a0a0', weight: 15 },
  { emoji: '🧲', type: 'buff', buff: 'magnet', color: '#a855f7', weight: 3 },
  { emoji: '⏱️', type: 'buff', buff: 'slow', color: '#3b82f6', weight: 3 },
  { emoji: '🛡️', type: 'buff', buff: 'shield', color: '#10b981', weight: 3 },
]

function pickItem() {
  const total = itemTypes.reduce((s, t) => s + t.weight, 0)
  let r = Math.random() * total
  for (const t of itemTypes) {
    r -= t.weight
    if (r <= 0) return t
  }
  return itemTypes[0]
}

function initStars() {
  stars = Array.from({ length: 50 }, () => ({
    x: Math.random() * w,
    y: Math.random() * h,
    size: Math.random() * 1.5 + 0.5,
    speed: Math.random() * 0.5 + 0.1
  }))
}

function initCanvasSize() {
  const cvs = canvasRef.value
  if (!cvs) return
  w = cvs.width = Math.min(window.innerWidth - 40, 400)
  h = cvs.height = Math.min(window.innerHeight - 150, 750)
  canvasRect = cvs.getBoundingClientRect() // 初始化时获取一次边界
}

onMounted(() => {
  initCanvasSize()
  // 监听窗口大小改变，重置缓存的边界
  window.addEventListener('resize', initCanvasSize)
})

onUnmounted(() => {
  if (animId) cancelAnimationFrame(animId)
  window.removeEventListener('resize', initCanvasSize)
})

function startGame() {
  // ✨ 修复2：如果已经有动画在运行，先掐断它，防止两套物理引擎叠加！
  if (animId) cancelAnimationFrame(animId)

  gameState.value = 'playing'
  score.value = 0
  lives.value = 5
  combo.value = 0
  level.value = 1
  frameCount = 0
  items = []
  particles = []
  floatingTexts = []
  activeBuffs = { magnet: 0, slow: 0, shield: false }

  initCanvasSize()
  paddle.x = w / 2 - paddle.width / 2
  initStars()
  gameLoop()
}

function addParticles(x, y, color, isBomb = false) {
  for (let i = 0; i < (isBomb ? 12 : 6); i++) {
    particles.push({
      x, y,
      vx: (Math.random() - 0.5) * (isBomb ? 8 : 5),
      vy: -Math.random() * (isBomb ? 5 : 3) - 1,
      life: 1,
      size: Math.random() * 3 + 2,
      color,
      type: 'dot'
    })
  }
  particles.push({ x, y, life: 1, size: 10, color, type: 'ring' })
}

function addFloatingText(x, y, text, color) {
  floatingTexts.push({ x, y, text, color, life: 1 })
}

function triggerShake(intensity = 10) {
  shakeAmount = intensity
}

function handleDamage() {
  if (activeBuffs.shield) {
    activeBuffs.shield = false
    addFloatingText(paddle.x + paddle.width/2, h - 50, "护盾抵挡!", "#10b981")
    return
  }
  combo.value = 0
  lives.value--
  triggerShake(12)
  if (lives.value <= 0) gameOver()
}

function gameOver() {
  gameState.value = 'over'
  if (score.value > bestScore.value) {
    bestScore.value = score.value
    localStorage.setItem('catchBest', String(score.value))
  }
}

function gameLoop() {
  if (gameState.value !== 'playing') return
  ctx = canvasRef.value.getContext('2d')

  ctx.save()
  if (shakeAmount > 0) {
    ctx.translate((Math.random() - 0.5) * shakeAmount, (Math.random() - 0.5) * shakeAmount)
    shakeAmount *= 0.9
    if (shakeAmount < 0.5) shakeAmount = 0
  }

  ctx.clearRect(0, 0, w, h)
  frameCount++

  ctx.fillStyle = 'rgba(255, 255, 255, 0.5)'
  stars.forEach(s => {
    s.y += s.speed * (activeBuffs.slow > 0 ? 0.3 : 1)
    if (s.y > h) { s.y = 0; s.x = Math.random() * w }
    ctx.beginPath()
    ctx.arc(s.x, s.y, s.size, 0, Math.PI * 2)
    ctx.fill()
  })

  if (activeBuffs.magnet > 0) activeBuffs.magnet--
  if (activeBuffs.slow > 0) activeBuffs.slow--

  level.value = Math.floor(score.value / 1000) + 1
  const spawnRate = Math.max(20, 55 - level.value * 4)

  if (frameCount % spawnRate === 0) {
    const type = pickItem()
    items.push({
      x: Math.random() * (w - 30) + 15,
      y: -20,
      speed: (2.5 + Math.random() * 1.5 + level.value * 0.2),
      size: type.type === 'buff' ? 26 : 22,
      ...type,
      wobble: Math.random() * Math.PI * 2
    })
  }

  paddle.x += (mouseX - paddle.width / 2 - paddle.x) * 0.3
  paddle.x = Math.max(0, Math.min(w - paddle.width, paddle.x))

  const paddleY = h - 45
  ctx.shadowColor = activeBuffs.shield ? '#10b981' : '#ff6b9d'
  ctx.shadowBlur = 15
  ctx.fillStyle = activeBuffs.shield ? 'rgba(16, 185, 129, 0.8)' : 'rgba(255, 107, 157, 0.8)'
  ctx.beginPath()
  ctx.roundRect(paddle.x, paddleY, paddle.width, paddle.height, 6)
  ctx.fill()
  ctx.shadowBlur = 0
  ctx.fillStyle = '#fff'
  ctx.beginPath()
  ctx.roundRect(paddle.x + 2, paddleY + 2, paddle.width - 4, paddle.height - 4, 4)
  ctx.fill()

  ctx.font = '22px sans-serif'
  ctx.fillText('🧺', paddle.x + paddle.width / 2 - 14, paddleY - 8)

  const speedMult = activeBuffs.slow > 0 ? 0.4 : 1
  items = items.filter(item => {
    if (activeBuffs.magnet > 0 && item.type !== 'bomb') {
      const dx = (paddle.x + paddle.width/2) - item.x
      const dy = paddleY - item.y
      const dist = Math.sqrt(dx*dx + dy*dy)
      if (dist < 150) {
        item.x += dx * 0.04
        item.y += dy * 0.04
      }
    }

    item.y += item.speed * speedMult
    item.wobble += 0.05
    const wobbleX = Math.sin(item.wobble) * (item.type === 'buff' ? 0 : 6)

    ctx.shadowColor = item.color
    ctx.shadowBlur = 10
    ctx.font = `${item.size}px sans-serif`
    ctx.fillText(item.emoji, item.x + wobbleX - item.size / 2, item.y)
    ctx.shadowBlur = 0

    const caught = item.y >= paddleY - item.size/2 && item.y <= paddleY + paddle.height &&
        (item.x + wobbleX) > paddle.x - 10 && (item.x + wobbleX) < paddle.x + paddle.width + 10

    if (caught) {
      if (item.type === 'buff') {
        if (item.buff === 'magnet') activeBuffs.magnet = 300
        if (item.buff === 'slow') activeBuffs.slow = 300
        if (item.buff === 'shield') activeBuffs.shield = true
        addFloatingText(item.x, item.y, "Buff Up!", item.color)
        addParticles(item.x, item.y, item.color)
      } else if (item.type === 'score') {
        combo.value++
        const multiplier = 1 + (combo.value * 0.1)
        const finalScore = Math.floor(item.score * multiplier)
        score.value += finalScore
        addFloatingText(item.x, item.y, `+${finalScore}`, item.color)
        addParticles(item.x, item.y, item.color)
      } else if (item.type === 'bomb') {
        score.value = Math.max(0, score.value + item.score)
        addFloatingText(item.x, item.y, `${item.score}`, item.color)
        addParticles(item.x, item.y, item.color, true)
        handleDamage()
      }
      return false
    }

    if (item.y > h + 20) {
      if (item.type === 'score') {
        addFloatingText(item.x, h - 20, "Miss", "#ff4444")
        handleDamage()
      }
      return false
    }
    return true
  })

  // ✨ 修复1：使用 Math.max 确保透明度和半径永远不会小于 0，彻底防止 Canvas 报错卡死！
  particles.forEach(p => {
    if (p.type === 'dot') {
      p.x += p.vx
      p.y += p.vy
      p.vy += 0.15
      p.life -= 0.02

      const safeLife = Math.max(0, p.life)
      const safeSize = Math.max(0, p.size * safeLife)

      ctx.beginPath()
      ctx.arc(p.x, p.y, safeSize, 0, Math.PI * 2)
      ctx.fillStyle = p.color
      ctx.globalAlpha = safeLife
      ctx.fill()
    } else if (p.type === 'ring') {
      p.size += 1.5
      p.life -= 0.05

      const safeLife = Math.max(0, p.life)

      ctx.beginPath()
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2)
      ctx.strokeStyle = p.color
      ctx.lineWidth = Math.max(0.1, 2 * safeLife)
      ctx.globalAlpha = safeLife
      ctx.stroke()
    }
    ctx.globalAlpha = 1
  })
  particles = particles.filter(p => p.life > 0)

  floatingTexts.forEach(ft => {
    ft.y -= 1
    ft.life -= 0.02

    const safeLife = Math.max(0, ft.life)

    ctx.fillStyle = ft.color
    ctx.globalAlpha = safeLife
    ctx.font = 'bold 16px sans-serif'
    ctx.fillText(ft.text, ft.x - 10, ft.y)
    ctx.globalAlpha = 1
  })
  floatingTexts = floatingTexts.filter(ft => ft.life > 0)

  ctx.restore()

  ctx.fillStyle = '#fff'
  ctx.font = 'bold 16px sans-serif'
  ctx.fillText(`Score: ${score.value}`, 15, 30)
  ctx.font = '12px sans-serif'
  ctx.fillText(`Lv.${level.value}`, 15, 50)

  ctx.font = '16px sans-serif'
  ctx.fillText('❤️'.repeat(lives.value), w - lives.value * 18 - 10, 30)

  if (combo.value > 1) {
    const comboScale = Math.min(1 + combo.value * 0.05, 1.5)
    ctx.fillStyle = `hsl(${Math.min(combo.value * 5, 60)}, 100%, 50%)`
    ctx.font = `bold ${14 * comboScale}px sans-serif`
    ctx.fillText(`${combo.value} Combo!`, w / 2 - 40, 40)
  }

  let buffX = w / 2 - 30
  if (activeBuffs.magnet > 0) { ctx.fillText('🧲', buffX, 70); buffX += 25 }
  if (activeBuffs.slow > 0) { ctx.fillText('⏱️', buffX, 70); buffX += 25 }
  if (activeBuffs.shield) { ctx.fillText('🛡️', buffX, 70) }

  animId = requestAnimationFrame(gameLoop)
}

function handleMove(e) {
  if (gameState.value !== 'playing') return
  // ✨ 修复2补充：直接使用缓存的边界 rect，大幅提升每秒 120 次鼠标事件的性能
  if (!canvasRect && canvasRef.value) canvasRect = canvasRef.value.getBoundingClientRect()

  const clientX = e.touches ? e.touches[0].clientX : e.clientX
  const left = canvasRect ? canvasRect.left : 0
  mouseX = clientX - left
}
</script>
<template>
  <div class="catch-game">
    <div class="header">
      <h2 class="section-title">爱心接接乐 ✨</h2>
      <p class="game-desc">收集爱心与道具，挑战最高连击，小心心碎！</p>
    </div>

    <div class="game-area"
         @mousemove="handleMove"
         @touchmove.prevent="handleMove"
         @touchstart.prevent="handleMove"
    >
      <canvas ref="canvasRef"></canvas>

      <transition name="fade">
        <div v-if="gameState === 'ready'" class="game-overlay glass-panel">
          <div class="logo">🧺</div>
          <h3 class="title">准备迎接爱意</h3>

          <div class="guide-box">
            <div class="guide-row"><span>💖💕⭐💎</span> 加分</div>
            <div class="guide-row"><span>💔</span> 减分&扣血</div>
            <div class="guide-row"><span>🧲 ⏱️ 🛡️</span> 超级道具</div>
          </div>

          <p class="tip">提示: 漏接好的爱心也会扣除生命，注意保持连击倍率！</p>
          <button class="game-btn pulse-btn" @click.stop="startGame">
            <span>开始游戏</span>
          </button>

          <div v-if="bestScore > 0" class="best-score-badge">
            🏆 历史最高: {{ bestScore }}
          </div>
        </div>
      </transition>

      <transition name="fade">
        <div v-if="gameState === 'over'" class="game-overlay glass-panel">
          <div class="over-emoji">😭</div>
          <h3 class="title">游戏结束</h3>

          <div class="score-card">
            <div class="stat">
              <span class="stat-label">本次得分</span>
              <span class="stat-val highlight">{{ score }}</span>
            </div>
            <div class="divider"></div>
            <div class="stat">
              <span class="stat-label">达到等级</span>
              <span class="stat-val">Lv.{{ level }}</span>
            </div>
          </div>

          <button class="game-btn" @click.stop="startGame">
            <span>再来一局</span>
          </button>
        </div>
      </transition>
    </div>
  </div>
</template>

<style scoped>
.catch-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 76px);
  padding: 30px 15px;
  background: radial-gradient(circle at top, #1a1a2e, #10101c);
  font-family: 'PingFang SC', system-ui, sans-serif;
}

.header { text-align: center; margin-bottom: 20px; }

.section-title {
  font-size: 1.8rem;
  font-weight: 800;
  margin: 0 0 8px 0;
  background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 99%, #fecfef 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 2px 10px rgba(255, 154, 158, 0.3);
}

.game-desc {
  font-size: 0.9rem;
  color: #a0a0b0;
  margin: 0;
}

.game-area {
  position: relative;
  border-radius: 24px;
  overflow: hidden;
  background: #0f0c29;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5), inset 0 0 0 1px rgba(255,255,255,0.1);
  touch-action: none;
}

canvas { display: block; }

/* 覆盖层与毛玻璃 UI */
.game-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  z-index: 10;
}

.glass-panel {
  background: rgba(15, 12, 41, 0.7);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}

.logo, .over-emoji { font-size: 56px; filter: drop-shadow(0 4px 10px rgba(0,0,0,0.5)); margin-bottom: 10px; }
.title { color: #fff; font-size: 1.5rem; margin: 0 0 20px 0; font-weight: 700; letter-spacing: 1px; }

.guide-box {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 15px 25px;
  margin-bottom: 15px;
}
.guide-row { color: #d0d0e0; font-size: 0.9rem; margin: 8px 0; display: flex; justify-content: space-between; gap: 20px; }
.guide-row span { letter-spacing: 3px; }

.tip { font-size: 0.8rem; color: #ff8c8c; text-align: center; max-width: 80%; margin-bottom: 25px; line-height: 1.4; }

/* 得分卡片 */
.score-card {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 20px;
  padding: 20px 40px;
  margin-bottom: 30px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.2);
}
.divider { width: 1px; height: 40px; background: rgba(255,255,255,0.1); margin: 0 30px; }
.stat { display: flex; flex-direction: column; align-items: center; }
.stat-label { font-size: 0.8rem; color: #a0a0b0; margin-bottom: 6px; }
.stat-val { font-size: 1.4rem; font-weight: 700; color: #fff; }
.stat-val.highlight { color: #ff6b9d; font-size: 2rem; text-shadow: 0 0 10px rgba(255, 107, 157, 0.5); }

/* 按钮样式 */
.game-btn {
  position: relative;
  padding: 14px 40px;
  border-radius: 30px;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  border: none;
  color: #fff;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  box-shadow: 0 4px 15px rgba(196, 77, 255, 0.4);
}
.game-btn::after {
  content: ''; position: absolute; top: 0; left: -100%; width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.4), transparent);
  transition: all 0.5s;
}
.game-btn:hover { transform: translateY(-3px); box-shadow: 0 8px 25px rgba(196, 77, 255, 0.6); }
.game-btn:hover::after { left: 100%; }

.pulse-btn { animation: pulse 2s infinite; }
@keyframes pulse {
  0% { box-shadow: 0 0 0 0 rgba(255, 107, 157, 0.6); }
  70% { box-shadow: 0 0 0 15px rgba(255, 107, 157, 0); }
  100% { box-shadow: 0 0 0 0 rgba(255, 107, 157, 0); }
}

.best-score-badge {
  margin-top: 20px;
  background: rgba(255, 215, 0, 0.15);
  color: #ffd700;
  padding: 6px 16px;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 600;
  border: 1px solid rgba(255, 215, 0, 0.3);
}

/* 过渡动画 */
.fade-enter-active, .fade-leave-active { transition: opacity 0.4s ease, transform 0.4s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: scale(0.95); }
</style>