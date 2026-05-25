<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'

const canvasRef = ref(null)
const gameState = ref('ready')
const score = ref(0)
const bestScore = ref(parseInt(localStorage.getItem('catBest') || '0'))
const coins = ref(0)
const hasDoubleJump = ref(false)
const jumpsLeft = ref(0)
const showPowerUp = ref('')

let ctx, w, h, animId
let cat = { x: 70, y: 0, vy: 0, size: 28, onGround: true }
let obstacles = []
let collectibles = []
let particles = []
let bgParticles = []
let frameCount = 0
let groundY = 0
let keys = {}
let speed = 3
let invincible = 0
let combo = 0

const distance = computed(() => Math.max(0, 100 - score.value))

function startGame() {
  gameState.value = 'playing'
  score.value = 0
  coins.value = 0
  frameCount = 0
  obstacles = []
  collectibles = []
  particles = []
  speed = 3
  invincible = 0
  combo = 0
  hasDoubleJump.value = true
  jumpsLeft.value = 2
  showPowerUp.value = ''
  const cvs = canvasRef.value
  w = cvs.width = Math.min(window.innerWidth - 40, 520)
  h = cvs.height = 340
  groundY = h - 60
  cat = { x: 70, y: groundY - 28, vy: 0, size: 28, onGround: true }

  bgParticles = Array.from({ length: 20 }, () => ({
    x: Math.random() * w,
    y: Math.random() * groundY,
    r: Math.random() * 1.5 + 0.5,
    speed: Math.random() * 0.5 + 0.2
  }))

  gameLoop()
}

function addParticles(x, y, color, count = 5) {
  for (let i = 0; i < count; i++) {
    particles.push({
      x, y,
      vx: (Math.random() - 0.5) * 4,
      vy: -Math.random() * 3 - 1,
      life: 1,
      size: Math.random() * 4 + 1,
      color
    })
  }
}

function jump() {
  if (gameState.value !== 'playing') return
  if (cat.onGround) {
    cat.vy = -12
    cat.onGround = false
    jumpsLeft.value = 1
    addParticles(cat.x, cat.y + cat.size, 'rgba(255, 107, 157, 0.8)', 4)
  } else if (jumpsLeft.value > 0) {
    cat.vy = -10
    jumpsLeft.value--
    addParticles(cat.x, cat.y + cat.size, 'rgba(196, 77, 255, 0.8)', 6)
    showPowerUp.value = '二段跳!'
    setTimeout(() => { showPowerUp.value = '' }, 600)
  }
}

function gameLoop() {
  if (gameState.value !== 'playing') return
  ctx = canvasRef.value.getContext('2d')
  ctx.clearRect(0, 0, w, h)
  frameCount++

  // Background particles
  bgParticles.forEach(p => {
    p.x -= p.speed
    if (p.x < 0) { p.x = w; p.y = Math.random() * groundY }
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(255, 255, 255, 0.15)`
    ctx.fill()
  })

  // Ground
  const grd = ctx.createLinearGradient(0, groundY, w, groundY)
  grd.addColorStop(0, 'rgba(255, 107, 157, 0.4)')
  grd.addColorStop(0.5, 'rgba(196, 77, 255, 0.4)')
  grd.addColorStop(1, 'rgba(102, 126, 234, 0.4)')
  ctx.fillStyle = grd
  ctx.fillRect(0, groundY, w, 2)

  // Ground pattern
  for (let i = 0; i < w; i += 20) {
    const xPos = ((i - frameCount * speed * 0.5) % w + w) % w
    ctx.beginPath()
    ctx.arc(xPos, groundY + 15 + Math.sin(xPos * 0.05) * 3, 1, 0, Math.PI * 2)
    ctx.fillStyle = 'rgba(255, 255, 255, 0.08)'
    ctx.fill()
  }

  speed = 3 + frameCount * 0.003

  // Spawn obstacles
  if (frameCount % Math.max(40, 75 - Math.floor(frameCount / 40)) === 0) {
    const type = Math.random()
    if (type < 0.3 && frameCount > 200) {
      // Tall thin obstacle
      obstacles.push({ x: w + 20, width: 14, height: 50 + Math.random() * 20, type: 'tall' })
    } else if (type < 0.5 && frameCount > 400) {
      // Double obstacle
      obstacles.push({ x: w + 20, width: 18, height: 30, type: 'normal' })
      obstacles.push({ x: w + 60, width: 18, height: 35, type: 'normal' })
    } else {
      obstacles.push({ x: w + 20, width: 18 + Math.random() * 14, height: 28 + Math.random() * 22, type: 'normal' })
    }
  }

  // Spawn collectibles
  if (frameCount % 60 === 0 && Math.random() > 0.3) {
    const type = Math.random() < 0.15 ? 'star' : 'heart'
    collectibles.push({
      x: w + 20,
      y: groundY - 60 - Math.random() * 80,
      type,
      size: type === 'star' ? 18 : 14,
      collected: false
    })
  }

  // Update obstacles
  obstacles.forEach(ob => { ob.x -= speed })
  obstacles = obstacles.filter(ob => ob.x > -50)

  // Update collectibles
  collectibles.forEach(c => { c.x -= speed })
  collectibles = collectibles.filter(c => c.x > -50 && !c.collected)

  // Draw obstacles
  obstacles.forEach(ob => {
    const gradient = ctx.createLinearGradient(ob.x, groundY - ob.height, ob.x, groundY)
    gradient.addColorStop(0, 'rgba(255, 80, 100, 0.8)')
    gradient.addColorStop(1, 'rgba(255, 50, 80, 0.5)')
    ctx.shadowColor = 'rgba(255, 80, 80, 0.5)'
    ctx.shadowBlur = 10
    ctx.fillStyle = gradient
    ctx.beginPath()
    const r = 4
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

  // Draw collectibles
  collectibles.forEach(c => {
    if (c.collected) return
    ctx.font = `${c.size}px serif`
    const bob = Math.sin(frameCount * 0.08 + c.x * 0.01) * 4
    if (c.type === 'star') {
      ctx.shadowColor = 'rgba(255, 215, 0, 0.6)'
      ctx.shadowBlur = 8
      ctx.fillText('⭐', c.x - c.size / 2, c.y + bob)
    } else {
      ctx.shadowColor = 'rgba(255, 107, 157, 0.6)'
      ctx.shadowBlur = 6
      ctx.fillText('💖', c.x - c.size / 2, c.y + bob)
    }
    ctx.shadowBlur = 0
  })

  // Jump input
  if (keys['ArrowUp'] || keys[' '] || keys['w'] || keys['W']) {
    if (!keys._jumped) {
      jump()
      keys._jumped = true
    }
  } else {
    keys._jumped = false
  }

  // Physics
  cat.vy += 0.6
  cat.y += cat.vy
  if (cat.y >= groundY - cat.size) {
    cat.y = groundY - cat.size
    cat.vy = 0
    cat.onGround = true
    jumpsLeft.value = 2
  }

  // Particles update
  particles.forEach(p => {
    p.x += p.vx
    p.y += p.vy
    p.vy += 0.12
    p.life -= 0.025
  })
  particles = particles.filter(p => p.life > 0)
  particles.forEach(p => {
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.size * p.life, 0, Math.PI * 2)
    ctx.fillStyle = p.color.replace('0.8', String(p.life))
    ctx.fill()
  })

  // Draw cat
  if (invincible > 0) {
    invincible--
    ctx.globalAlpha = Math.sin(frameCount * 0.5) * 0.3 + 0.7
  }
  ctx.shadowColor = 'rgba(255, 200, 100, 0.4)'
  ctx.shadowBlur = 14
  ctx.font = `${cat.size}px serif`
  ctx.fillText('🐱', cat.x - cat.size / 2, cat.y + cat.size - 5)
  ctx.shadowBlur = 0
  ctx.globalAlpha = 1

  // Trail effect
  if (!cat.onGround) {
    ctx.beginPath()
    ctx.arc(cat.x - 5, cat.y + cat.size - 2, 3, 0, Math.PI * 2)
    ctx.fillStyle = 'rgba(255, 107, 157, 0.3)'
    ctx.fill()
  }

  // Collect items
  collectibles.forEach(c => {
    if (c.collected) return
    const dx = cat.x - c.x
    const dy = (cat.y + cat.size / 2) - c.y
    if (Math.abs(dx) < 24 && Math.abs(dy) < 24) {
      c.collected = true
      if (c.type === 'star') {
        coins.value += 5
        combo++
        addParticles(c.x, c.y, 'rgba(255, 215, 0, 0.9)', 8)
        showPowerUp.value = combo > 2 ? `连击x${combo}! +${5 + combo}` : '+5'
      } else {
        coins.value += 1
        addParticles(c.x, c.y, 'rgba(255, 107, 157, 0.9)', 5)
        showPowerUp.value = '+1'
      }
      setTimeout(() => { showPowerUp.value = '' }, 500)
    }
  })

  // Partner at end
  ctx.font = '28px serif'
  ctx.fillText('💕', w - 55, groundY - 12)

  // HUD
  ctx.fillStyle = 'rgba(255, 255, 255, 0.5)'
  ctx.font = '12px sans-serif'
  ctx.fillText(`💖 ${coins.value}`, 12, 24)
  ctx.fillText(`${distance.value} 步`, w - 55, 24)

  // Combo indicator
  if (combo > 2) {
    ctx.fillStyle = 'rgba(255, 215, 0, 0.6)'
    ctx.font = '10px sans-serif'
    ctx.fillText(`combo x${combo}`, 12, 42)
  }

  // Collision
  if (invincible <= 0) {
    for (const ob of obstacles) {
      if (
        cat.x + cat.size / 2 - 8 > ob.x &&
        cat.x - cat.size / 2 + 8 < ob.x + ob.width &&
        cat.y + cat.size > groundY - ob.height + 5
      ) {
        gameState.value = 'over'
        const total = score.value + coins.value
        if (total > bestScore.value) {
          bestScore.value = total
          localStorage.setItem('catBest', String(total))
        }
        return
      }
    }
  }

  score.value = Math.floor(frameCount / 10)
  if (score.value >= 100) {
    gameState.value = 'won'
    const total = score.value + coins.value
    if (total > bestScore.value) {
      bestScore.value = total
      localStorage.setItem('catBest', String(total))
    }
    return
  }

  // Reset combo if on ground for too long
  if (cat.onGround && frameCount % 60 === 0) combo = 0

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
  if (gameState.value === 'playing') jump()
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
    <p class="game-desc">空格/↑/W 跳跃（支持二段跳！），收集爱心和星星，躲过障碍去见她</p>

    <div class="game-area" @click="handleTap" @touchstart.prevent="handleTap">
      <canvas ref="canvasRef"></canvas>

      <transition name="pop">
        <div v-if="showPowerUp" class="power-up-text">{{ showPowerUp }}</div>
      </transition>

      <div v-if="gameState === 'ready'" class="game-overlay">
        <div class="overlay-scene">
          <span class="scene-cat">🐱</span>
          <span class="scene-items">💖 ⭐ 💖</span>
          <span class="scene-heart">💕</span>
        </div>
        <h3>帮小猫跑过去见她</h3>
        <div class="game-tips">
          <span>💖 = 1分</span>
          <span>⭐ = 5分</span>
          <span>连续收集有连击加成</span>
        </div>
        <button class="game-btn" @click.stop="startGame">开始游戏</button>
        <p v-if="bestScore > 0" class="best-score">最高分：{{ bestScore }}</p>
      </div>

      <div v-if="gameState === 'won'" class="game-overlay won">
        <div class="won-emoji">🐱💕</div>
        <h3>终于见到你啦！</h3>
        <p class="won-sub">跨过所有障碍，只为到你身边</p>
        <div class="final-score">
          <span>距离 {{ score }}</span>
          <span>收集 {{ coins }}</span>
          <span class="total">总分 {{ score + coins }}</span>
        </div>
        <button class="game-btn" @click.stop="startGame">再来一次</button>
      </div>

      <div v-if="gameState === 'over'" class="game-overlay over">
        <div class="over-emoji">😿</div>
        <h3>差一点就到了...</h3>
        <div class="final-score">
          <span>跑了 {{ score }} 步</span>
          <span>收集 {{ coins }} 分</span>
          <span class="total">总分 {{ score + coins }}</span>
        </div>
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
  max-width: 400px;
}

.game-area {
  position: relative;
  border-radius: 20px;
  overflow: hidden;
  background: linear-gradient(180deg, rgba(10, 10, 30, 0.8), rgba(15, 10, 35, 0.9));
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}

canvas { display: block; }

.power-up-text {
  position: absolute;
  top: 50px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1.1rem;
  font-weight: 600;
  color: #ffd700;
  text-shadow: 0 0 10px rgba(255, 215, 0, 0.5);
  pointer-events: none;
}

.pop-enter-active { transition: all 0.2s ease-out; }
.pop-leave-active { transition: all 0.3s ease-in; }
.pop-enter-from { opacity: 0; transform: translateX(-50%) scale(0.5) translateY(10px); }
.pop-leave-to { opacity: 0; transform: translateX(-50%) translateY(-20px); }

.game-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(10, 10, 26, 0.92);
  backdrop-filter: blur(8px);
  gap: 10px;
}

.overlay-scene {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;
}

.scene-cat { font-size: 32px; animation: bounce 1s ease-in-out infinite; }
.scene-items { font-size: 18px; color: rgba(255, 255, 255, 0.5); letter-spacing: 4px; }
.scene-heart { font-size: 28px; animation: pulse2 1.5s ease-in-out infinite; }

@keyframes bounce { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-6px); } }
@keyframes pulse2 { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.15); } }

.game-overlay h3 { color: rgba(255, 255, 255, 0.9); font-size: 1.1rem; font-weight: 500; }
.game-overlay.won h3 { color: #ff6b9d; font-size: 1.3rem; }

.game-tips {
  display: flex;
  gap: 12px;
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.4);
  margin: 4px 0;
}

.won-emoji, .over-emoji { font-size: 44px; margin-bottom: 4px; }
.won-sub { font-size: 0.85rem; color: rgba(255, 255, 255, 0.5); }

.final-score {
  display: flex;
  gap: 16px;
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.5);
  margin: 8px 0;
}

.final-score .total {
  color: #ffd700;
  font-weight: 600;
}

.best-score { font-size: 0.75rem; color: rgba(255, 215, 0, 0.5); margin-top: 4px; }

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
