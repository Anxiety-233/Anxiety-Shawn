<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const canvasRef = ref(null)
const gameState = ref('ready')
const score = ref(0)

let ctx, w, h, animId
let cat = { x: 60, y: 0, vy: 0, size: 30 }
let obstacles = []
let partner = { x: 0, y: 0 }
let frameCount = 0
let groundY = 0
let keys = {}

function startGame() {
  gameState.value = 'playing'
  score.value = 0
  frameCount = 0
  obstacles = []
  const cvs = canvasRef.value
  w = cvs.width = Math.min(window.innerWidth - 40, 500)
  h = cvs.height = 300
  groundY = h - 50
  cat = { x: 60, y: groundY - 30, vy: 0, size: 30, onGround: true }
  partner = { x: w - 60, y: groundY - 30 }
  gameLoop()
}

function gameLoop() {
  if (gameState.value !== 'playing') return
  ctx = canvasRef.value.getContext('2d')
  ctx.clearRect(0, 0, w, h)
  frameCount++

  ctx.fillStyle = 'rgba(255, 255, 255, 0.2)'
  ctx.fillRect(0, groundY, w, 2)

  if (frameCount % 80 === 0) {
    obstacles.push({
      x: w,
      width: 20 + Math.random() * 15,
      height: 25 + Math.random() * 20
    })
  }

  obstacles.forEach(ob => { ob.x -= 3 })
  obstacles = obstacles.filter(ob => ob.x > -50)

  obstacles.forEach(ob => {
    ctx.fillStyle = 'rgba(255, 100, 100, 0.6)'
    ctx.fillRect(ob.x, groundY - ob.height, ob.width, ob.height)
    ctx.fillStyle = 'rgba(255, 100, 100, 0.3)'
    ctx.fillRect(ob.x - 2, groundY - ob.height - 2, ob.width + 4, ob.height + 4)
  })

  if (keys['ArrowUp'] || keys[' ']) {
    if (cat.onGround) {
      cat.vy = -10
      cat.onGround = false
    }
  }

  cat.vy += 0.5
  cat.y += cat.vy
  if (cat.y >= groundY - cat.size) {
    cat.y = groundY - cat.size
    cat.vy = 0
    cat.onGround = true
  }

  // Draw cat
  ctx.font = `${cat.size}px serif`
  ctx.fillText('🐱', cat.x - cat.size / 2, cat.y + cat.size - 5)

  // Draw partner
  ctx.font = '30px serif'
  ctx.fillText('💕', partner.x - 15, partner.y + 20)

  // Collision detection
  for (const ob of obstacles) {
    if (
      cat.x + cat.size / 2 > ob.x &&
      cat.x - cat.size / 2 < ob.x + ob.width &&
      cat.y + cat.size > groundY - ob.height
    ) {
      gameState.value = 'over'
      return
    }
  }

  // Win condition
  score.value = Math.floor(frameCount / 10)
  if (score.value >= 50) {
    gameState.value = 'won'
    return
  }

  animId = requestAnimationFrame(gameLoop)
}

function handleKeyDown(e) {
  keys[e.key] = true
}

function handleKeyUp(e) {
  keys[e.key] = false
}

function handleTap() {
  if (gameState.value === 'playing' && cat.onGround) {
    cat.vy = -10
    cat.onGround = false
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
    <p class="game-desc">按 空格/↑ 跳跃（手机点击屏幕），躲过障碍去见她！</p>

    <div class="game-area">
      <canvas ref="canvasRef" @click="handleTap"></canvas>

      <div v-if="gameState === 'ready'" class="game-overlay">
        <p class="overlay-emoji">🐱 → 💕</p>
        <button class="game-btn" @click="startGame">开始游戏</button>
      </div>

      <div v-if="gameState === 'won'" class="game-overlay won">
        <p class="overlay-emoji">🐱💕</p>
        <h3>终于见到你啦！</h3>
        <button class="game-btn" @click="startGame">再来一次</button>
      </div>

      <div v-if="gameState === 'over'" class="game-overlay">
        <p class="overlay-emoji">😿</p>
        <h3>差一点就到了...</h3>
        <p class="over-score">跑了 {{ score }} 步</p>
        <button class="game-btn" @click="startGame">再试一次</button>
      </div>
    </div>

    <div v-if="gameState === 'playing'" class="score-display">
      距离：{{ 50 - score }} 步
    </div>
  </div>
</template>

<style scoped>
.cat-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 70px);
  padding: 40px 20px;
}

.section-title {
  font-size: 1.5rem;
  margin-bottom: 12px;
  color: rgba(255, 255, 255, 0.9);
}

.game-desc {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.5);
  margin-bottom: 32px;
  text-align: center;
}

.game-area {
  position: relative;
  border-radius: 16px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.1);
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
  background: rgba(10, 10, 26, 0.85);
  backdrop-filter: blur(5px);
  gap: 12px;
}

.game-overlay.won h3 {
  color: #ff6b9d;
  font-size: 1.3rem;
}

.game-overlay h3 {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1.1rem;
}

.overlay-emoji {
  font-size: 36px;
  margin-bottom: 8px;
}

.over-score {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.5);
}

.game-btn {
  padding: 10px 28px;
  border-radius: 20px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: #fff;
  font-size: 0.9rem;
  margin-top: 8px;
  transition: all 0.3s;
}

.game-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}

.score-display {
  margin-top: 16px;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.6);
}
</style>
