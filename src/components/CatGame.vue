<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const canvasRef = ref(null)
const gameState = ref('ready')
const score = ref(0)
const bestScore = ref(0)

let ctx, w, h, animId
let cat = { x: 60, y: 0, vy: 0, size: 30 }
let obstacles = []
let particles = []
let frameCount = 0
let groundY = 0
let keys = {}
let speed = 3

function startGame() {
  gameState.value = 'playing'
  score.value = 0
  frameCount = 0
  obstacles = []
  particles = []
  speed = 3
  const cvs = canvasRef.value
  w = cvs.width = Math.min(window.innerWidth - 40, 500)
  h = cvs.height = 320
  groundY = h - 60
  cat = { x: 70, y: groundY - 30, vy: 0, size: 28, onGround: true }
  gameLoop()
}

function addParticle(x, y) {
  for (let i = 0; i < 3; i++) {
    particles.push({
      x, y,
      vx: (Math.random() - 0.5) * 2,
      vy: -Math.random() * 2,
      life: 1,
      size: Math.random() * 3 + 1
    })
  }
}

function gameLoop() {
  if (gameState.value !== 'playing') return
  ctx = canvasRef.value.getContext('2d')
  ctx.clearRect(0, 0, w, h)
  frameCount++

  // Ground line with gradient
  const grd = ctx.createLinearGradient(0, groundY, w, groundY)
  grd.addColorStop(0, 'rgba(255, 107, 157, 0.3)')
  grd.addColorStop(0.5, 'rgba(196, 77, 255, 0.3)')
  grd.addColorStop(1, 'rgba(102, 126, 234, 0.3)')
  ctx.fillStyle = grd
  ctx.fillRect(0, groundY, w, 2)

  // Ground dots
  for (let i = 0; i < w; i += 30) {
    ctx.beginPath()
    ctx.arc((i - frameCount * speed) % w + w, groundY + 20, 1, 0, Math.PI * 2)
    ctx.fillStyle = 'rgba(255, 255, 255, 0.1)'
    ctx.fill()
  }

  // Increase speed over time
  speed = 3 + frameCount * 0.002

  if (frameCount % Math.max(50, 80 - Math.floor(frameCount / 50)) === 0) {
    obstacles.push({
      x: w + 20,
      width: 18 + Math.random() * 14,
      height: 28 + Math.random() * 22
    })
  }

  obstacles.forEach(ob => { ob.x -= speed })
  obstacles = obstacles.filter(ob => ob.x > -50)

  // Draw obstacles with glow
  obstacles.forEach(ob => {
    ctx.shadowColor = 'rgba(255, 80, 80, 0.4)'
    ctx.shadowBlur = 8
    ctx.fillStyle = 'rgba(255, 100, 120, 0.7)'
    const r = 4
    ctx.beginPath()
    ctx.moveTo(ob.x + r, groundY - ob.height)
    ctx.lineTo(ob.x + ob.width - r, groundY - ob.height)
    ctx.quadraticCurveTo(ob.x + ob.width, groundY - ob.height, ob.x + ob.width, groundY - ob.height + r)
    ctx.lineTo(ob.x + ob.width, groundY)
    ctx.lineTo(ob.x, groundY)
    ctx.lineTo(ob.x, groundY - ob.height + r)
    ctx.quadraticCurveTo(ob.x, groundY - ob.height, ob.x + r, groundY - ob.height)
    ctx.fill()
    ctx.shadowBlur = 0
  })

  // Jump
  if (keys['ArrowUp'] || keys[' '] || keys['w'] || keys['W']) {
    if (cat.onGround) {
      cat.vy = -11
      cat.onGround = false
      addParticle(cat.x, cat.y + cat.size)
    }
  }

  cat.vy += 0.55
  cat.y += cat.vy
  if (cat.y >= groundY - cat.size) {
    cat.y = groundY - cat.size
    cat.vy = 0
    cat.onGround = true
  }

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
    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(255, 107, 157, ${p.life})`
    ctx.fill()
  })

  // Draw cat with shadow
  ctx.shadowColor = 'rgba(255, 200, 100, 0.3)'
  ctx.shadowBlur = 12
  ctx.font = `${cat.size}px serif`
  ctx.fillText('🐱', cat.x - cat.size / 2, cat.y + cat.size - 5)
  ctx.shadowBlur = 0

  // Draw partner at end
  ctx.font = '28px serif'
  const partnerX = w - 50
  ctx.fillText('💕', partnerX - 14, groundY - 10)

  // Distance indicator
  ctx.fillStyle = 'rgba(255, 255, 255, 0.4)'
  ctx.font = '11px sans-serif'
  ctx.fillText(`${Math.max(0, 50 - score.value)} 步`, w - 60, 24)

  // Collision
  for (const ob of obstacles) {
    if (
      cat.x + cat.size / 2 - 6 > ob.x &&
      cat.x - cat.size / 2 + 6 < ob.x + ob.width &&
      cat.y + cat.size > groundY - ob.height + 4
    ) {
      gameState.value = 'over'
      if (score.value > bestScore.value) bestScore.value = score.value
      return
    }
  }

  score.value = Math.floor(frameCount / 10)
  if (score.value >= 50) {
    gameState.value = 'won'
    if (score.value > bestScore.value) bestScore.value = score.value
    return
  }

  animId = requestAnimationFrame(gameLoop)
}

function handleKeyDown(e) {
  keys[e.key] = true
  if (e.key === ' ') e.preventDefault()
}

function handleKeyUp(e) {
  keys[e.key] = false
}

function handleTap() {
  if (gameState.value === 'playing' && cat.onGround) {
    cat.vy = -11
    cat.onGround = false
    addParticle(cat.x, cat.y + cat.size)
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
  window.addEventListener('keyup', handleKeyUp)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
  window.removeEventListener('keyup', handleKeyUp)
  cancelAnimationFrame(animId)
})
</script>

<template>
  <div class="cat-game">
    <h2 class="section-title">小猫去见你</h2>
    <p class="game-desc">按 空格/↑/W 跳跃（手机点击画面），躲过障碍去见她！</p>

    <div class="game-area" @click="handleTap">
      <canvas ref="canvasRef"></canvas>

      <div v-if="gameState === 'ready'" class="game-overlay">
        <div class="overlay-scene">
          <span class="scene-cat">🐱</span>
          <span class="scene-dots">· · · · ·</span>
          <span class="scene-heart">💕</span>
        </div>
        <h3>帮小猫跑过去见她</h3>
        <button class="game-btn" @click.stop="startGame">开始游戏</button>
        <p v-if="bestScore > 0" class="best-score">最远记录：{{ bestScore }} 步</p>
      </div>

      <div v-if="gameState === 'won'" class="game-overlay won">
        <div class="won-emoji">🐱💕</div>
        <h3>终于见到你啦！</h3>
        <p class="won-sub">跨过所有障碍，只为到你身边</p>
        <button class="game-btn" @click.stop="startGame">再来一次</button>
      </div>

      <div v-if="gameState === 'over'" class="game-overlay over">
        <div class="over-emoji">😿</div>
        <h3>差一点就到了...</h3>
        <p class="over-score">跑了 {{ score }} 步</p>
        <button class="game-btn" @click.stop="startGame">再试一次</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.cat-game {
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
  cursor: pointer;
}

canvas {
  display: block;
}

.game-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(10, 10, 26, 0.9);
  backdrop-filter: blur(8px);
  gap: 12px;
}

.overlay-scene {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.scene-cat {
  font-size: 32px;
  animation: bounce 1s ease-in-out infinite;
}

.scene-dots {
  color: rgba(255, 255, 255, 0.3);
  letter-spacing: 4px;
}

.scene-heart {
  font-size: 28px;
  animation: pulse2 1.5s ease-in-out infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-6px); }
}

@keyframes pulse2 {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.15); }
}

.game-overlay h3 {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1.1rem;
  font-weight: 500;
}

.game-overlay.won h3 {
  color: #ff6b9d;
  font-size: 1.3rem;
}

.won-emoji, .over-emoji {
  font-size: 44px;
  margin-bottom: 4px;
}

.won-sub {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.5);
}

.over-score {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.4);
}

.best-score {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.3);
  margin-top: 4px;
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
