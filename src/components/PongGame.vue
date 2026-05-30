<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import Peer from 'peerjs'

const status = ref('idle')
const myId = ref('')
const remoteId = ref('')
const inputPeerId = ref('')
const score = ref({ me: 0, opponent: 0 })
const role = ref('')
const gameStarted = ref(false)
const canvas = ref(null)
const message = ref('')

let peer = null
let conn = null
let animFrame = null
let lastTime = 0

const CANVAS_W = 400
const CANVAS_H = 560
const PADDLE_W = 80
const PADDLE_H = 12
const BALL_R = 8
const PADDLE_SPEED = 7
const WIN_SCORE = 5

const state = {
  ball: { x: CANVAS_W / 2, y: CANVAS_H / 2, vx: 3, vy: 3 },
  myPaddle: CANVAS_W / 2,
  opPaddle: CANVAS_W / 2,
  keys: { left: false, right: false }
}

function initPeer() {
  status.value = 'connecting'
  const isSecure = window.location.protocol === 'https:'
  peer = new Peer({
    host: 'glossiest-samuel-complemental.ngrok-free.dev',
    port: isSecure ? 443 : 80,
    path: '/',
    secure: isSecure
  })

  peer.on('open', (id) => {
    myId.value = id
    status.value = 'waiting'
  })

  peer.on('connection', (c) => {
    conn = c
    remoteId.value = c.peer
    role.value = 'host'
    setupConn()
  })

  peer.on('error', (err) => {
    message.value = '连接错误: ' + err.type
    status.value = 'idle'
  })
}

function connectTo() {
  if (!inputPeerId.value.trim()) return
  conn = peer.connect(inputPeerId.value.trim())
  remoteId.value = inputPeerId.value.trim()
  role.value = 'guest'
  setupConn()
}

function setupConn() {
  conn.on('open', () => {
    status.value = 'connected'
    message.value = '已连接！准备开始游戏'
    startGame()
  })
  conn.on('data', (data) => {
    if (data.type === 'paddle') {
      state.opPaddle = data.x
    } else if (data.type === 'ball') {
      state.ball = data.ball
    } else if (data.type === 'score') {
      score.value = data.score
    }
  })
  conn.on('close', () => {
    status.value = 'waiting'
    gameStarted.value = false
    message.value = '对方已断开连接'
    cancelAnimationFrame(animFrame)
  })
}

function startGame() {
  gameStarted.value = true
  score.value = { me: 0, opponent: 0 }
  resetBall()
  lastTime = performance.now()
  gameLoop()
}

function resetBall() {
  state.ball.x = CANVAS_W / 2
  state.ball.y = CANVAS_H / 2
  state.ball.vx = (Math.random() > 0.5 ? 1 : -1) * 3
  state.ball.vy = (Math.random() > 0.5 ? 1 : -1) * 3
}

function gameLoop() {
  update()
  draw()
  animFrame = requestAnimationFrame(gameLoop)
}

function update() {
  if (state.keys.left) state.myPaddle = Math.max(PADDLE_W / 2, state.myPaddle - PADDLE_SPEED)
  if (state.keys.right) state.myPaddle = Math.min(CANVAS_W - PADDLE_W / 2, state.myPaddle + PADDLE_SPEED)

  if (conn && conn.open) {
    conn.send({ type: 'paddle', x: state.myPaddle })
  }

  if (role.value === 'host') {
    const b = state.ball
    b.x += b.vx
    b.y += b.vy

    if (b.x - BALL_R <= 0 || b.x + BALL_R >= CANVAS_W) b.vx *= -1

    const myY = CANVAS_H - 30
    if (b.vy > 0 && b.y + BALL_R >= myY && b.y + BALL_R <= myY + PADDLE_H) {
      if (b.x >= state.myPaddle - PADDLE_W / 2 && b.x <= state.myPaddle + PADDLE_W / 2) {
        b.vy *= -1.05
        b.vx += (b.x - state.myPaddle) * 0.1
      }
    }

    const opY = 20
    if (b.vy < 0 && b.y - BALL_R <= opY + PADDLE_H && b.y - BALL_R >= opY) {
      if (b.x >= state.opPaddle - PADDLE_W / 2 && b.x <= state.opPaddle + PADDLE_W / 2) {
        b.vy *= -1.05
        b.vx += (b.x - state.opPaddle) * 0.1
      }
    }

    const maxV = 8
    b.vx = Math.max(-maxV, Math.min(maxV, b.vx))
    b.vy = Math.max(-maxV, Math.min(maxV, b.vy))

    if (b.y < 0) {
      score.value.me++
      resetBall()
    } else if (b.y > CANVAS_H) {
      score.value.opponent++
      resetBall()
    }

    if (conn && conn.open) {
      conn.send({ type: 'ball', ball: { ...b } })
      conn.send({ type: 'score', score: { me: score.value.opponent, opponent: score.value.me } })
    }

    if (score.value.me >= WIN_SCORE || score.value.opponent >= WIN_SCORE) {
      gameStarted.value = false
      cancelAnimationFrame(animFrame)
      message.value = score.value.me >= WIN_SCORE ? '你赢了！🎉' : '对方赢了！'
    }
  }
}

function draw() {
  const ctx = canvas.value?.getContext('2d')
  if (!ctx) return

  ctx.fillStyle = '#1a1a2e'
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H)

  ctx.setLineDash([8, 8])
  ctx.strokeStyle = 'rgba(255,255,255,0.15)'
  ctx.beginPath()
  ctx.moveTo(0, CANVAS_H / 2)
  ctx.lineTo(CANVAS_W, CANVAS_H / 2)
  ctx.stroke()
  ctx.setLineDash([])

  const grad = ctx.createRadialGradient(state.ball.x, state.ball.y, 0, state.ball.x, state.ball.y, BALL_R * 2)
  grad.addColorStop(0, '#ff6b9d')
  grad.addColorStop(1, 'rgba(255,107,157,0)')
  ctx.fillStyle = grad
  ctx.fillRect(state.ball.x - BALL_R * 2, state.ball.y - BALL_R * 2, BALL_R * 4, BALL_R * 4)

  ctx.fillStyle = '#ff6b9d'
  ctx.beginPath()
  ctx.arc(state.ball.x, state.ball.y, BALL_R, 0, Math.PI * 2)
  ctx.fill()

  const myY = CANVAS_H - 30
  ctx.fillStyle = '#4fc3f7'
  ctx.shadowColor = '#4fc3f7'
  ctx.shadowBlur = 10
  ctx.beginPath()
  ctx.roundRect(state.myPaddle - PADDLE_W / 2, myY, PADDLE_W, PADDLE_H, 6)
  ctx.fill()

  ctx.fillStyle = '#ce93d8'
  ctx.shadowColor = '#ce93d8'
  ctx.beginPath()
  ctx.roundRect(state.opPaddle - PADDLE_W / 2, 20, PADDLE_W, PADDLE_H, 6)
  ctx.fill()
  ctx.shadowBlur = 0
}

function onKeyDown(e) {
  if (e.key === 'ArrowLeft' || e.key === 'a') state.keys.left = true
  if (e.key === 'ArrowRight' || e.key === 'd') state.keys.right = true
}
function onKeyUp(e) {
  if (e.key === 'ArrowLeft' || e.key === 'a') state.keys.left = false
  if (e.key === 'ArrowRight' || e.key === 'd') state.keys.right = false
}

let touchStartX = 0
function onTouchStart(e) {
  touchStartX = e.touches[0].clientX
}
function onTouchMove(e) {
  const dx = e.touches[0].clientX - touchStartX
  touchStartX = e.touches[0].clientX
  state.myPaddle = Math.max(PADDLE_W / 2, Math.min(CANVAS_W - PADDLE_W / 2, state.myPaddle + dx * 0.8))
}

function copyId() {
  navigator.clipboard.writeText(myId.value)
  message.value = '已复制ID！'
  setTimeout(() => { if (message.value === '已复制ID！') message.value = '' }, 2000)
}

onMounted(() => {
  initPeer()
  window.addEventListener('keydown', onKeyDown)
  window.addEventListener('keyup', onKeyUp)
})

onUnmounted(() => {
  window.removeEventListener('keydown', onKeyDown)
  window.removeEventListener('keyup', onKeyUp)
  cancelAnimationFrame(animFrame)
  if (peer) peer.destroy()
})
</script>

<template>
  <div class="pong-container">
    <h2 class="title">🏓 双人弹球对战</h2>
    <p class="subtitle">通过P2P连接和对方实时对战！</p>

    <div v-if="message" class="message">{{ message }}</div>

    <div v-if="!gameStarted" class="lobby">
      <div v-if="myId" class="id-box">
        <span class="label">你的ID：</span>
        <code class="peer-id">{{ myId }}</code>
        <button class="copy-btn" @click="copyId">复制</button>
      </div>

      <div v-if="status === 'waiting'" class="connect-box">
        <p class="hint">把你的ID发给对方，或输入对方的ID来连接：</p>
        <div class="input-row">
          <input v-model="inputPeerId" placeholder="输入对方的ID" @keyup.enter="connectTo" />
          <button class="connect-btn" @click="connectTo">连接</button>
        </div>
      </div>

      <div v-if="status === 'connecting'" class="loading">正在连接服务器...</div>
    </div>

    <div v-if="gameStarted" class="game-area">
      <div class="scoreboard">
        <span class="score-me">我 {{ score.me }}</span>
        <span class="score-sep">:</span>
        <span class="score-op">{{ score.opponent }} 对方</span>
      </div>
      <canvas
        ref="canvas"
        :width="CANVAS_W"
        :height="CANVAS_H"
        class="game-canvas"
        @touchstart="onTouchStart"
        @touchmove.prevent="onTouchMove"
      />
      <p class="controls-hint">⬅️ A/← 移动 ➡️ D/→ | 手机触屏滑动</p>
    </div>
  </div>
</template>

<style scoped>
.pong-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  color: #fff;
}
.title {
  font-size: 1.6rem;
  margin-bottom: 4px;
  background: linear-gradient(135deg, #4fc3f7, #ce93d8);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
.subtitle {
  color: rgba(255,255,255,0.5);
  font-size: 0.85rem;
  margin-bottom: 20px;
}
.message {
  background: rgba(255,107,157,0.15);
  border: 1px solid rgba(255,107,157,0.3);
  border-radius: 8px;
  padding: 8px 16px;
  margin-bottom: 16px;
  font-size: 0.9rem;
}
.lobby {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  width: 100%;
  max-width: 400px;
}
.id-box {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(255,255,255,0.05);
  padding: 10px 16px;
  border-radius: 10px;
  border: 1px solid rgba(255,255,255,0.1);
  flex-wrap: wrap;
  justify-content: center;
}
.peer-id {
  color: #4fc3f7;
  font-size: 0.8rem;
  word-break: break-all;
}
.copy-btn {
  padding: 4px 12px;
  border-radius: 6px;
  background: rgba(79,195,247,0.2);
  border: 1px solid rgba(79,195,247,0.4);
  color: #4fc3f7;
  cursor: pointer;
  font-size: 0.8rem;
}
.connect-box {
  width: 100%;
  text-align: center;
}
.hint {
  color: rgba(255,255,255,0.6);
  font-size: 0.85rem;
  margin-bottom: 10px;
}
.input-row {
  display: flex;
  gap: 8px;
}
.input-row input {
  flex: 1;
  padding: 10px 14px;
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.15);
  background: rgba(255,255,255,0.05);
  color: #fff;
  font-size: 0.9rem;
  outline: none;
}
.input-row input:focus {
  border-color: rgba(79,195,247,0.5);
}
.connect-btn {
  padding: 10px 20px;
  border-radius: 8px;
  background: linear-gradient(135deg, #4fc3f7, #ce93d8);
  border: none;
  color: #fff;
  font-weight: 600;
  cursor: pointer;
}
.loading {
  color: rgba(255,255,255,0.6);
}
.game-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}
.scoreboard {
  font-size: 1.4rem;
  font-weight: 700;
  display: flex;
  gap: 12px;
}
.score-me { color: #4fc3f7; }
.score-sep { color: rgba(255,255,255,0.3); }
.score-op { color: #ce93d8; }
.game-canvas {
  border-radius: 12px;
  border: 1px solid rgba(255,255,255,0.1);
  max-width: 100%;
  touch-action: none;
}
.controls-hint {
  color: rgba(255,255,255,0.4);
  font-size: 0.75rem;
}
</style>
