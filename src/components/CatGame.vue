<script setup>
import { ref, onMounted, onUnmounted, computed, nextTick } from 'vue'

const canvasRef = ref(null)
const gameState = ref('ready') // ready, playing, won, over
const score = ref(0)
const bestScore = ref(parseInt(localStorage.getItem('catBest') || '0'))
const coins = ref(0)
const jumpsLeft = ref(0)
const showPowerUp = ref('')

// ✨ 核心游戏变量
let ctx, w, h, animId
let cat = { x: 80, y: 0, vy: 0, size: 32, onGround: true, rotation: 0 }
let obstacles = []
let collectibles = []
let particles = []
let stars = [] // 视差背景星空
let frameCount = 0
let groundY = 0
let keys = {}
let baseSpeed = 3.5
let currentSpeed = 3.5
let shakeAmount = 0

// ✨ 连击与狂热模式系统
let combo = 0
const feverThreshold = 10
const isFever = ref(false)
let feverTimer = 0

const distance = computed(() => Math.max(0, 100 - score.value))
const comboProgress = computed(() => Math.min(combo / feverThreshold * 100, 100))

function startGame() {
  gameState.value = 'playing'
  score.value = 0
  coins.value = 0
  frameCount = 0
  obstacles = []
  collectibles = []
  particles = []
  combo = 0
  isFever.value = false
  feverTimer = 0
  jumpsLeft.value = 2
  showPowerUp.value = ''

  initCanvas()
  initBackground()

  cat = { x: 80, y: groundY - 32, vy: 0, size: 32, onGround: true, rotation: 0 }
  gameLoop()
}

function initCanvas() {
  const cvs = canvasRef.value
  if (!cvs) return
  w = cvs.width = Math.min(window.innerWidth - 20, 600)
  h = cvs.height = Math.min(window.innerHeight - 150, 400)
  groundY = h - 50
}

function initBackground() {
  stars = Array.from({ length: 30 }, () => ({
    x: Math.random() * w,
    y: Math.random() * groundY,
    size: Math.random() * 1.5 + 0.5,
    speed: Math.random() * 0.2 + 0.05
  }))
}

// 屏幕震动
function triggerShake(intensity = 10) { shakeAmount = intensity }

// 粒子爆炸系统
function addParticles(x, y, color, count = 6, speedMult = 1) {
  for (let i = 0; i < count; i++) {
    particles.push({
      x, y,
      vx: (Math.random() - 0.5) * 6 * speedMult,
      vy: -Math.random() * 5 * speedMult - 2,
      life: 1,
      size: Math.random() * 5 + 2,
      color,
      type: Math.random() > 0.5 ? 'circle' : 'star'
    })
  }
}

// 触发狂热模式
function activateFever() {
  isFever.value = true
  feverTimer = 300 // 维持约 5 秒
  showPowerUp.value = '🔥 FEVER MODE! 🔥'
  triggerShake(5)
  addParticles(cat.x, cat.y, 'rgba(255, 215, 0, 1)', 20, 2)
  setTimeout(() => { showPowerUp.value = '' }, 1000)
}

function jump() {
  if (gameState.value !== 'playing') return
  if (cat.onGround) {
    cat.vy = -12.5
    cat.onGround = false
    jumpsLeft.value = 1
    addParticles(cat.x, cat.y + cat.size, 'rgba(255, 107, 157, 0.8)', 5)
  } else if (jumpsLeft.value > 0) {
    cat.vy = -10.5
    jumpsLeft.value--
    // 二段跳特效
    addParticles(cat.x, cat.y + cat.size, 'rgba(196, 77, 255, 0.9)', 8)
    cat.rotation = -Math.PI / 4 // 翻滚动作
  }
}

function gameLoop() {
  if (gameState.value !== 'playing') return
  ctx = canvasRef.value.getContext('2d')

  // 震屏处理
  ctx.save()
  if (shakeAmount > 0) {
    ctx.translate((Math.random() - 0.5) * shakeAmount, (Math.random() - 0.5) * shakeAmount)
    shakeAmount *= 0.9
    if (shakeAmount < 0.5) shakeAmount = 0
  }

  ctx.clearRect(0, 0, w, h)
  frameCount++

  // --- 1. 背景绘制 (视差星空与巨大月亮) ---
  ctx.fillStyle = 'rgba(255, 255, 255, 0.6)'
  stars.forEach(s => {
    s.x -= s.speed * (isFever.value ? 2 : 1)
    if (s.x < 0) { s.x = w; s.y = Math.random() * groundY }
    ctx.beginPath()
    ctx.arc(s.x, s.y, s.size, 0, Math.PI * 2)
    ctx.fill()
  })

  // 绘制月亮
  ctx.shadowColor = 'rgba(255, 236, 210, 0.3)'
  ctx.shadowBlur = 30
  ctx.fillStyle = '#ffecd2'
  ctx.beginPath()
  ctx.arc(w - 80, 80, 40, 0, Math.PI * 2)
  ctx.fill()
  ctx.shadowBlur = 0

  // --- 2. 地面绘制 ---
  const grd = ctx.createLinearGradient(0, groundY, 0, h)
  grd.addColorStop(0, isFever.value ? 'rgba(255, 215, 0, 0.5)' : 'rgba(196, 77, 255, 0.4)')
  grd.addColorStop(1, 'rgba(10, 10, 26, 0.8)')
  ctx.fillStyle = grd
  ctx.fillRect(0, groundY, w, h - groundY)

  // 地面网格流动光线 (赛博朋克感)
  ctx.strokeStyle = isFever.value ? 'rgba(255, 215, 0, 0.2)' : 'rgba(255, 255, 255, 0.1)'
  ctx.lineWidth = 2
  for (let i = 0; i < w; i += 40) {
    const xPos = ((i - frameCount * currentSpeed) % w + w) % w
    ctx.beginPath()
    ctx.moveTo(xPos, groundY)
    ctx.lineTo(xPos - 30, h)
    ctx.stroke()
  }

  // --- 3. 游戏数值与难度更新 ---
  baseSpeed = 4 + frameCount * 0.001
  currentSpeed = isFever.value ? baseSpeed * 1.6 : baseSpeed // 狂热模式提速

  if (isFever.value) {
    feverTimer--
    if (feverTimer <= 0) {
      isFever.value = false
      combo = 0 // 狂热结束清空连击
    }
  }

  // 生成障碍物 (霓虹地刺)
  if (frameCount % Math.max(45, 80 - Math.floor(frameCount / 60)) === 0) {
    const type = Math.random()
    if (type < 0.2 && frameCount > 300) {
      obstacles.push({ x: w + 20, width: 20, height: 60 + Math.random() * 20, type: 'tall' })
    } else if (type < 0.4 && frameCount > 500) {
      obstacles.push({ x: w + 20, width: 20, height: 35, type: 'double' })
      obstacles.push({ x: w + 70, width: 20, height: 40, type: 'double' })
    } else {
      obstacles.push({ x: w + 20, width: 22 + Math.random() * 10, height: 35 + Math.random() * 15, type: 'normal' })
    }
  }

  // 生成收集物
  if (frameCount % 50 === 0 && Math.random() > 0.2) {
    const type = Math.random() < 0.2 ? 'star' : 'heart'
    collectibles.push({
      x: w + 30,
      y: groundY - 70 - Math.random() * 80,
      type,
      size: type === 'star' ? 22 : 18,
      collected: false
    })
  }

  // --- 4. 物理与渲染 ---
  // 更新与绘制障碍物
  obstacles.forEach(ob => {
    ob.x -= currentSpeed

    // 绘制发光地刺
    ctx.shadowColor = isFever.value ? 'rgba(100, 100, 100, 0.5)' : 'rgba(255, 50, 100, 0.8)'
    ctx.shadowBlur = 15
    ctx.fillStyle = isFever.value ? 'rgba(100, 100, 100, 0.3)' : 'rgba(255, 50, 100, 0.9)'

    ctx.beginPath()
    ctx.moveTo(ob.x, groundY)
    ctx.lineTo(ob.x + ob.width/2, groundY - ob.height)
    ctx.lineTo(ob.x + ob.width, groundY)
    ctx.fill()
    ctx.shadowBlur = 0
  })
  obstacles = obstacles.filter(ob => ob.x > -50)

  // 更新与绘制收集物
  collectibles.forEach(c => {
    c.x -= currentSpeed
    if (c.collected) return
    ctx.font = `${c.size}px sans-serif`
    const bob = Math.sin(frameCount * 0.1 + c.x * 0.05) * 6
    if (c.type === 'star') {
      ctx.shadowColor = 'rgba(255, 215, 0, 0.8)'; ctx.shadowBlur = 15
      ctx.fillText('⭐', c.x - c.size / 2, c.y + bob)
    } else {
      ctx.shadowColor = 'rgba(255, 107, 157, 0.8)'; ctx.shadowBlur = 10
      ctx.fillText('💖', c.x - c.size / 2, c.y + bob)
    }
    ctx.shadowBlur = 0
  })
  collectibles = collectibles.filter(c => c.x > -50 && !c.collected)

  // 按键输入
  if (keys['ArrowUp'] || keys[' '] || keys['w'] || keys['W']) {
    if (!keys._jumped) { jump(); keys._jumped = true }
  } else { keys._jumped = false }

  // 小猫物理引擎
  cat.vy += 0.7 // 重力
  cat.y += cat.vy
  if (cat.y >= groundY - cat.size) {
    cat.y = groundY - cat.size
    cat.vy = 0
    cat.onGround = true
    jumpsLeft.value = 2
    cat.rotation = 0
  } else {
    // 空中旋转回正逻辑
    if (cat.rotation < 0) cat.rotation += 0.05
  }

  // ✨ 核心修复：安全处理粒子渲染，防止出现负数尺寸导致 Canvas 崩溃
  particles.forEach(p => {
    p.x += p.vx; p.y += p.vy; p.vy += 0.15; p.life -= 0.03

    // 使用 Math.max 确保生命值和尺寸绝不为负数
    const safeLife = Math.max(0, p.life)

    ctx.globalAlpha = safeLife
    ctx.fillStyle = p.color
    ctx.beginPath()
    if (p.type === 'circle') {
      ctx.arc(p.x, p.y, p.size * safeLife, 0, Math.PI*2)
    } else {
      ctx.fillRect(p.x, p.y, p.size * safeLife, p.size * safeLife)
    }
    ctx.fill()
  })
  ctx.globalAlpha = 1
  particles = particles.filter(p => p.life > 0)

  // --- 5. 绘制小猫 (挤压与拉伸 Q弹效果) ---
  ctx.save()
  ctx.translate(cat.x, cat.y + cat.size / 2) // 移动到小猫中心点
  ctx.rotate(cat.rotation)

  // Squash and Stretch based on Y velocity
  const stretch = Math.max(0.6, Math.min(1.4, 1 + cat.vy * 0.03))
  const squash = 1 / stretch
  ctx.scale(squash, stretch)

  // 狂热模式无敌光环
  if (isFever.value) {
    ctx.shadowColor = '#ffd700'
    ctx.shadowBlur = 20 + Math.sin(frameCount * 0.5) * 10

    // 狂热残影
    if (frameCount % 3 === 0) {
      addParticles(cat.x - 10, cat.y + 10, `hsl(${frameCount % 360}, 100%, 50%)`, 1, 0.5)
    }
  }

  ctx.font = `${cat.size}px sans-serif`
  ctx.fillText('🐱', -cat.size / 2, cat.size / 2 - 5)
  ctx.restore()

  // --- 6. 碰撞检测与逻辑 ---
  // 收集品判断
  collectibles.forEach(c => {
    if (c.collected) return
    const dx = cat.x - c.x
    const dy = (cat.y + cat.size / 2) - c.y
    if (Math.abs(dx) < 30 && Math.abs(dy) < 30) {
      c.collected = true

      if (c.type === 'star') {
        coins.value += 5
        addParticles(c.x, c.y, 'rgba(255, 215, 0, 1)', 10)
      } else {
        coins.value += 1
        addParticles(c.x, c.y, 'rgba(255, 107, 157, 1)', 6)
      }

      if (!isFever.value) {
        combo++
        if (combo >= feverThreshold) activateFever()
      }
    }
  })

  // 落地断连击
  if (cat.onGround && frameCount % 60 === 0 && !isFever.value) {
    combo = Math.max(0, combo - 1)
  }

  // 障碍物撞击判断 (Hitbox 缩减，增加容错率)
  for (let i = obstacles.length - 1; i >= 0; i--) {
    const ob = obstacles[i]
    if (
        cat.x + cat.size / 3 > ob.x &&
        cat.x - cat.size / 3 < ob.x + ob.width &&
        cat.y + cat.size > groundY - ob.height + 8
    ) {
      if (isFever.value) {
        // 狂热模式直接撞碎障碍！
        addParticles(ob.x + ob.width/2, groundY - ob.height/2, 'rgba(255, 255, 255, 0.8)', 15, 2)
        triggerShake(5)
        score.value += 5
        obstacles.splice(i, 1) // 移除障碍
      } else {
        // 普通状态死亡
        triggerShake(15)
        ctx.restore() // 恢复之前保存的 context
        gameOver()
        return
      }
    }
  }

  // 终点检测
  score.value = Math.floor(frameCount / 15)
  if (score.value >= 100) {
    ctx.restore()
    gameWin()
    return
  }

  // 终点老婆图标
  const targetX = w - 40 - (distance.value <= 10 ? (10 - distance.value) * 10 : 0)
  ctx.font = '32px sans-serif'
  ctx.fillText('💕', targetX, groundY - 10)

  ctx.restore() // 恢复全局画布震动

  animId = requestAnimationFrame(gameLoop)
}

function gameOver() {
  gameState.value = 'over'
  saveScore()
}

function gameWin() {
  gameState.value = 'won'
  saveScore()
}

function saveScore() {
  const total = score.value + coins.value
  if (total > bestScore.value) {
    bestScore.value = total
    localStorage.setItem('catBest', String(total))
  }
}

// 事件绑定
function handleKeyDown(e) { keys[e.key] = true; if (e.key === ' ') e.preventDefault() }
function handleKeyUp(e) { keys[e.key] = false }
function handleTap(e) {
  if (e.target.tagName !== 'BUTTON' && gameState.value === 'playing') jump()
}

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
  window.addEventListener('keyup', handleKeyUp)
  initCanvas()
  window.addEventListener('resize', initCanvas)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
  window.removeEventListener('keyup', handleKeyUp)
  window.removeEventListener('resize', initCanvas)
  if (animId) cancelAnimationFrame(animId)
})
</script>

<template>
  <div class="cat-game">
    <h2 class="section-title">小猫去见你</h2>
    <p class="game-desc">跨越万水千山，只为奔向你身边</p>

    <div class="game-container">

      <div class="hud-panel" v-if="gameState === 'playing'">
        <div class="stats">
          <span class="score-badge">💖 {{ coins }}</span>
          <span class="dist-badge">距你 {{ distance }} 步</span>
        </div>

        <div class="fever-bar-bg" :class="{ 'fever-active': isFever }">
          <div class="fever-bar-fill" :style="{ width: isFever ? `${(feverTimer / 300) * 100}%` : `${comboProgress}%` }"></div>
          <span class="fever-text">{{ isFever ? 'FEVER RUSH!!!' : (combo > 0 ? `${combo} COMBO` : 'FEVER METER') }}</span>
        </div>
      </div>

      <div class="game-area" @mousedown="handleTap" @touchstart.prevent="handleTap">
        <canvas ref="canvasRef"></canvas>

        <transition name="pop">
          <div v-if="showPowerUp" class="power-up-text" :class="{'super-text': isFever}">{{ showPowerUp }}</div>
        </transition>

        <transition name="fade">
          <div v-if="gameState === 'ready'" class="game-overlay glass-panel">
            <div class="overlay-scene">
              <span class="scene-cat">🐱</span>
              <span class="scene-items">✨💨</span>
              <span class="scene-heart">💕</span>
            </div>
            <h3>操作指南</h3>
            <div class="guide-box">
              <p>👆 点击屏幕 / 空格键 <b>跳跃</b></p>
              <p>🪂 空中再次点击触发 <b>二段跳</b></p>
              <p>🔥 连续收集不落地可积攒 <b>狂热槽</b></p>
            </div>
            <button class="game-btn pulse-btn" @click.stop="startGame">
              <span>出发吧！</span>
            </button>
            <p v-if="bestScore > 0" class="best-score">👑 最高记录：{{ bestScore }} 分</p>
          </div>

          <div v-else-if="gameState === 'won'" class="game-overlay glass-panel win-panel">
            <div class="emoji-banner">🐱💞👩</div>
            <h3 class="win-title">终于见到你啦！</h3>
            <p class="win-sub">跨过所有障碍，只为到你身边</p>

            <div class="score-card">
              <div class="stat-col"><span>奔赴距离</span><strong>{{ score }}</strong></div>
              <div class="stat-line"></div>
              <div class="stat-col"><span>收集心意</span><strong>{{ coins }}</strong></div>
              <div class="stat-line"></div>
              <div class="stat-col highlight"><span>总分</span><strong>{{ score + coins }}</strong></div>
            </div>

            <button class="game-btn" @click.stop="startGame">再见你一次</button>
          </div>

          <div v-else-if="gameState === 'over'" class="game-overlay glass-panel over-panel">
            <div class="emoji-banner grayscale">😿💨</div>
            <h3 class="over-title">哎呀，绊倒了...</h3>

            <div class="score-card">
              <div class="stat-col"><span>跑了</span><strong>{{ score }}</strong></div>
              <div class="stat-line"></div>
              <div class="stat-col"><span>收集</span><strong>{{ coins }}</strong></div>
              <div class="stat-line"></div>
              <div class="stat-col highlight"><span>总计</span><strong>{{ score + coins }}</strong></div>
            </div>

            <button class="game-btn retry-btn" @click.stop="startGame">拍拍灰，再试一次</button>
          </div>
        </transition>
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
  padding: 30px 15px;
  background: radial-gradient(circle at top, #1a1525, #0a0a10);
  font-family: 'PingFang SC', sans-serif;
  overflow-x: hidden;
}

.section-title {
  font-size: 1.8rem;
  font-weight: 800;
  margin-bottom: 5px;
  background: linear-gradient(135deg, #fff, #ff9a9e);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 2px 5px rgba(255, 154, 158, 0.3));
}

.game-desc { font-size: 0.85rem; color: rgba(255, 255, 255, 0.5); margin-bottom: 20px; }

/* 游戏区域布局 */
.game-container { position: relative; width: 100%; max-width: 600px; }

/* HUD 数据显示 */
.hud-panel {
  position: absolute; top: 15px; left: 15px; right: 15px;
  display: flex; flex-direction: column; gap: 10px; z-index: 5; pointer-events: none;
}
.stats { display: flex; justify-content: space-between; }
.score-badge, .dist-badge {
  background: rgba(0, 0, 0, 0.4); border: 1px solid rgba(255,255,255,0.1);
  padding: 6px 12px; border-radius: 20px; font-size: 0.85rem; color: #fff; font-weight: 600;
  backdrop-filter: blur(4px);
}
.dist-badge { color: #a0c4ff; border-color: rgba(160, 196, 255, 0.3); }

/* Fever 槽 */
.fever-bar-bg {
  width: 150px; height: 16px; background: rgba(0, 0, 0, 0.5); border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.1); position: relative; overflow: hidden; margin: 0 auto;
}
.fever-bar-fill {
  height: 100%; background: linear-gradient(90deg, #ff9a9e, #fecfef); transition: width 0.2s ease-out;
}
.fever-text {
  position: absolute; width: 100%; text-align: center; top: 1px; left: 0;
  font-size: 0.65rem; font-weight: 900; color: #fff; text-shadow: 0 1px 2px #000; letter-spacing: 1px;
}
.fever-active { box-shadow: 0 0 15px rgba(255, 215, 0, 0.6); border-color: #ffd700; }
.fever-active .fever-bar-fill { background: linear-gradient(90deg, #ffd700, #ff5722); animation: feverBlink 0.2s infinite; }
@keyframes feverBlink { 0% { opacity: 0.8; } 100% { opacity: 1; } }

/* 画布区域 */
.game-area {
  position: relative; border-radius: 24px; overflow: hidden;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5), inset 0 0 0 1px rgba(255,255,255,0.1);
  background: #0a0a1a; cursor: pointer; touch-action: none;
}
canvas { display: block; width: 100%; }

/* 提示文字 */
.power-up-text {
  position: absolute; top: 30%; left: 50%; transform: translateX(-50%);
  font-size: 1.2rem; font-weight: 800; color: #fff; text-shadow: 0 2px 10px rgba(196, 77, 255, 0.8);
  pointer-events: none; z-index: 6;
}
.super-text { font-size: 1.6rem; color: #ffd700; text-shadow: 0 0 20px #ff5722; font-style: italic; }

/* 动画过渡 */
.pop-enter-active { transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.pop-leave-active { transition: all 0.3s ease-in; }
.pop-enter-from { opacity: 0; transform: translate(-50%, 20px) scale(0.5); }
.pop-leave-to { opacity: 0; transform: translate(-50%, -30px); }
.fade-enter-active, .fade-leave-active { transition: opacity 0.4s, transform 0.4s; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: scale(0.95); }

/* --- 毛玻璃覆盖层 UI --- */
.game-overlay {
  position: absolute; inset: 0; z-index: 10; display: flex; flex-direction: column;
  align-items: center; justify-content: center; padding: 20px; overflow-y: auto;
}
.glass-panel { background: rgba(15, 10, 25, 0.75); backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px); }

/* 场景装饰 */
.overlay-scene { display: flex; align-items: center; gap: 15px; margin-bottom: 10px; }
.scene-cat { font-size: 38px; animation: bounce 1s infinite; }
.scene-items { font-size: 20px; animation: slideIn 2s infinite linear; }
.scene-heart { font-size: 32px; animation: pulseBeat 1.5s infinite; filter: drop-shadow(0 0 10px rgba(255,107,157,0.5)); }

@keyframes bounce { 0%, 100% { transform: translateY(0) scale(1, 1); } 50% { transform: translateY(-10px) scale(0.95, 1.05); } }
@keyframes slideIn { 0% { transform: translateX(10px); opacity: 0; } 50% { opacity: 1; } 100% { transform: translateX(-10px); opacity: 0; } }
@keyframes pulseBeat { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.15); } }

.game-overlay h3 { font-size: 1.4rem; color: #fff; margin-bottom: 15px; font-weight: 700; letter-spacing: 1px; }
.guide-box {
  background: rgba(255, 255, 255, 0.05); border: 1px solid rgba(255,255,255,0.1);
  padding: 15px 20px; border-radius: 16px; margin-bottom: 20px; width: 85%;
}
.guide-box p { color: #d0d0e0; font-size: 0.85rem; margin: 8px 0; display: flex; align-items: center; gap: 8px; }
.guide-box b { color: #ff9a9e; }

/* 结算面板样式 */
.emoji-banner { font-size: 50px; margin-bottom: 5px; filter: drop-shadow(0 5px 15px rgba(0,0,0,0.4)); }
.grayscale { filter: grayscale(100%) opacity(0.8); }
.win-title { color: #ff6b9d; }
.over-title { color: #a0a0b0; }
.win-sub { font-size: 0.85rem; color: #d0d0e0; margin-top: -10px; margin-bottom: 20px; }

.score-card {
  display: flex; align-items: center; background: rgba(0,0,0,0.3); border-radius: 16px;
  padding: 15px 20px; margin-bottom: 25px; border: 1px solid rgba(255,255,255,0.05);
}
.stat-col { display: flex; flex-direction: column; align-items: center; width: 70px; }
.stat-col span { font-size: 0.7rem; color: #aaa; margin-bottom: 6px; }
.stat-col strong { font-size: 1.2rem; color: #fff; font-variant-numeric: tabular-nums; }
.stat-line { width: 1px; height: 30px; background: rgba(255,255,255,0.1); margin: 0 10px; }
.stat-col.highlight strong { color: #ffd700; font-size: 1.5rem; text-shadow: 0 0 10px rgba(255,215,0,0.4); }

.best-score { font-size: 0.8rem; color: rgba(255, 215, 0, 0.8); margin-top: 15px; font-weight: 600; }

/* 炫酷按钮 */
.game-btn {
  padding: 14px 45px; border-radius: 30px; border: none;
  background: linear-gradient(135deg, #ff6b9d, #c44dff); color: #fff;
  font-size: 1.05rem; font-weight: 700; cursor: pointer;
  box-shadow: 0 5px 20px rgba(196, 77, 255, 0.4); transition: all 0.3s;
}
.game-btn:hover { transform: translateY(-3px); box-shadow: 0 8px 25px rgba(196, 77, 255, 0.6); }
.retry-btn { background: linear-gradient(135deg, #667eea, #764ba2); box-shadow: 0 5px 20px rgba(102, 126, 234, 0.4); }
.retry-btn:hover { box-shadow: 0 8px 25px rgba(102, 126, 234, 0.6); }

.pulse-btn { animation: btnPulse 2s infinite; }
@keyframes btnPulse {
  0% { box-shadow: 0 0 0 0 rgba(255, 107, 157, 0.6); }
  70% { box-shadow: 0 0 0 15px rgba(255, 107, 157, 0); }
  100% { box-shadow: 0 0 0 0 rgba(255, 107, 157, 0); }
}
</style>