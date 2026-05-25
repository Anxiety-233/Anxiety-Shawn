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
let paddle = { x: 0, width: 60, height: 10 }
let items = []
let particles = []
let frameCount = 0
let mouseX = 0
let touching = false

const itemTypes = [
  { emoji: '💖', score: 1, color: 'rgba(255, 107, 157, 0.8)', weight: 40 },
  { emoji: '💕', score: 2, color: 'rgba(236, 64, 122, 0.8)', weight: 25 },
  { emoji: '⭐', score: 5, color: 'rgba(255, 215, 0, 0.8)', weight: 15 },
  { emoji: '💎', score: 10, color: 'rgba(100, 200, 255, 0.8)', weight: 5 },
  { emoji: '💔', score: -3, color: 'rgba(128, 128, 128, 0.8)', weight: 15 },
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

function startGame() {
  gameState.value = 'playing'
  score.value = 0
  lives.value = 5
  combo.value = 0
  level.value = 1
  frameCount = 0
  items = []
  particles = []
  const cvs = canvasRef.value
  w = cvs.width = Math.min(window.innerWidth - 40, 400)
  h = cvs.height = 450
  paddle.x = w / 2 - paddle.width / 2
  gameLoop()
}

function addParticles(x, y, color, count = 6) {
  for (let i = 0; i < count; i++) {
    particles.push({
      x, y,
      vx: (Math.random() - 0.5) * 5,
      vy: -Math.random() * 3 - 1,
      life: 1,
      size: Math.random() * 3 + 2,
      color
    })
  }
}

function gameLoop() {
  if (gameState.value !== 'playing') return
  ctx = canvasRef.value.getContext('2d')
  ctx.clearRect(0, 0, w, h)
  frameCount++

  level.value = Math.floor(score.value / 20) + 1
  const spawnRate = Math.max(25, 50 - level.value * 3)

  if (frameCount % spawnRate === 0) {
    const type = pickItem()
    items.push({
      x: Math.random() * (w - 30) + 15,
      y: -20,
      speed: 1.5 + Math.random() * 1 + level.value * 0.3,
      size: 22,
      ...type,
      wobble: Math.random() * Math.PI * 2
    })
  }

  // Update paddle position
  paddle.x = mouseX - paddle.width / 2
  paddle.x = Math.max(0, Math.min(w - paddle.width, paddle.x))

  // Draw paddle with glow
  const paddleGrad = ctx.createLinearGradient(paddle.x, h - 40, paddle.x + paddle.width, h - 40)
  paddleGrad.addColorStop(0, '#ff6b9d')
  paddleGrad.addColorStop(1, '#c44dff')
  ctx.shadowColor = 'rgba(255, 107, 157, 0.5)'
  ctx.shadowBlur = 12
  ctx.fillStyle = paddleGrad
  ctx.beginPath()
  ctx.roundRect(paddle.x, h - 40, paddle.width, paddle.height, 5)
  ctx.fill()
  ctx.shadowBlur = 0

  // Draw paddle character
  ctx.font = '20px serif'
  ctx.fillText('🧺', paddle.x + paddle.width / 2 - 10, h - 44)

  // Update & draw items
  items.forEach(item => {
    item.y += item.speed
    item.wobble += 0.05
    const wobbleX = Math.sin(item.wobble) * 8

    ctx.font = `${item.size}px serif`
    ctx.fillText(item.emoji, item.x + wobbleX - item.size / 2, item.y)
  })

  // Check catches
  items = items.filter(item => {
    const itemCenterX = item.x
    const caught = item.y >= h - 55 && item.y <= h - 25 &&
      itemCenterX > paddle.x - 10 && itemCenterX < paddle.x + paddle.width + 10

    if (caught) {
      if (item.score > 0) {
        const bonusScore = item.score + (combo.value > 3 ? Math.floor(combo.value / 3) : 0)
        score.value += bonusScore
        combo.value++
        addParticles(item.x, item.y, item.color, 8)
      } else {
        score.value = Math.max(0, score.value + item.score)
        combo.value = 0
        lives.value--
        addParticles(item.x, item.y, 'rgba(128, 128, 128, 0.8)', 4)
        if (lives.value <= 0) {
          gameState.value = 'over'
          if (score.value > bestScore.value) {
            bestScore.value = score.value
            localStorage.setItem('catchBest', String(score.value))
          }
        }
      }
      return false
    }

    if (item.y > h + 20) {
      if (item.score > 0) {
        combo.value = 0
        lives.value--
        if (lives.value <= 0) {
          gameState.value = 'over'
          if (score.value > bestScore.value) {
            bestScore.value = score.value
            localStorage.setItem('catchBest', String(score.value))
          }
        }
      }
      return false
    }
    return true
  })

  // Particles
  particles.forEach(p => {
    p.x += p.vx
    p.y += p.vy
    p.vy += 0.1
    p.life -= 0.03
  })
  particles = particles.filter(p => p.life > 0)
  particles.forEach(p => {
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.size * p.life, 0, Math.PI * 2)
    ctx.fillStyle = p.color
    ctx.globalAlpha = p.life
    ctx.fill()
    ctx.globalAlpha = 1
  })

  // HUD
  ctx.fillStyle = 'rgba(255, 255, 255, 0.6)'
  ctx.font = '12px sans-serif'
  ctx.fillText(`分数: ${score.value}`, 12, 22)
  ctx.fillText(`等级: ${level.value}`, 12, 40)

  // Lives
  ctx.fillText('❤️'.repeat(lives.value), w - lives.value * 18 - 8, 22)

  // Combo
  if (combo.value > 3) {
    ctx.fillStyle = 'rgba(255, 215, 0, 0.7)'
    ctx.font = '11px sans-serif'
    ctx.fillText(`🔥 连击 x${combo.value}`, w / 2 - 30, 22)
  }

  animId = requestAnimationFrame(gameLoop)
}

function handleMove(e) {
  if (gameState.value !== 'playing') return
  const rect = canvasRef.value.getBoundingClientRect()
  if (e.touches) {
    mouseX = e.touches[0].clientX - rect.left
  } else {
    mouseX = e.clientX - rect.left
  }
}

function handleTouch(e) {
  e.preventDefault()
  touching = true
  handleMove(e)
}

onMounted(() => {})
onUnmounted(() => { cancelAnimationFrame(animId) })
</script>

<template>
  <div class="catch-game">
    <h2 class="section-title">爱心接接乐</h2>
    <p class="game-desc">移动鼠标/手指接住掉落的爱心，小心碎心💔哦！</p>

    <div class="game-area"
      @mousemove="handleMove"
      @touchmove.prevent="handleMove"
      @touchstart.prevent="handleTouch"
    >
      <canvas ref="canvasRef"></canvas>

      <div v-if="gameState === 'ready'" class="game-overlay">
        <div class="item-guide">
          <span>💖 +1</span>
          <span>💕 +2</span>
          <span>⭐ +5</span>
          <span>💎 +10</span>
          <span>💔 -3且减命</span>
        </div>
        <h3>接住所有的爱</h3>
        <p class="tip">漏接好的也会减命哦，共5条命</p>
        <button class="game-btn" @click.stop="startGame">开始游戏</button>
        <p v-if="bestScore > 0" class="best-score">最高分：{{ bestScore }}</p>
      </div>

      <div v-if="gameState === 'over'" class="game-overlay">
        <div class="over-emoji">😿</div>
        <h3>游戏结束</h3>
        <div class="final-stats">
          <div class="stat">
            <span class="stat-val">{{ score }}</span>
            <span class="stat-label">最终得分</span>
          </div>
          <div class="stat">
            <span class="stat-val">Lv.{{ level }}</span>
            <span class="stat-label">达到等级</span>
          </div>
        </div>
        <button class="game-btn" @click.stop="startGame">再来一次</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.catch-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 76px);
  padding: 40px 20px;
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

.game-desc {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.4);
  margin-bottom: 32px;
  text-align: center;
}

.game-area {
  position: relative;
  border-radius: 20px;
  overflow: hidden;
  background: linear-gradient(180deg, rgba(10, 10, 30, 0.8), rgba(15, 10, 35, 0.9));
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.3);
  touch-action: none;
}

canvas { display: block; }

.game-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(10, 10, 26, 0.92);
  backdrop-filter: blur(8px);
  gap: 12px;
}

.item-guide {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.5);
  margin-bottom: 12px;
}

.game-overlay h3 {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1.2rem;
}

.tip {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.35);
}

.over-emoji { font-size: 44px; }

.final-stats {
  display: flex;
  gap: 32px;
  margin: 8px 0;
}

.stat {
  text-align: center;
}

.stat-val {
  display: block;
  font-size: 1.5rem;
  font-weight: 600;
  color: #ff6b9d;
}

.stat-label {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.4);
}

.best-score {
  font-size: 0.75rem;
  color: rgba(255, 215, 0, 0.5);
}

.game-btn {
  padding: 12px 32px;
  border-radius: 24px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: #fff;
  font-size: 0.9rem;
  margin-top: 8px;
  transition: all 0.3s;
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.3);
}

.game-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 24px rgba(102, 126, 234, 0.5);
}
</style>
