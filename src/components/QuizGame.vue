<script setup>
import { ref, computed, nextTick } from 'vue'
import gsap from 'gsap'

const allQuestions = [
  { q: '我最想和你一起做什么？', options: ['吃饭', '散步', '看电影', '什么都不做，只待在一起'], answer: 3 },
  { q: '如果可以送你一样东西？', options: ['星星', '一整天的陪伴', '一个永远的拥抱', '全世界'], answer: 3 },
  { q: '最喜欢什么时候的你？', options: ['笑的时候', '发呆的时候', '撒娇的时候', '每时每刻'], answer: 3 },
  { q: '我们最像哪对CP？', options: ['甜蜜日常型', '搞笑吵闹型', '安静陪伴型', '独一无二型'], answer: 3 },
  { q: '如果世界末日，我会？', options: ['带你逃跑', '抱紧你', '和你一起看最后的星空', '以上全部'], answer: 3 },
  { q: '你在我心里是什么？', options: ['小太阳', '避风港', '全世界', '以上都是'], answer: 3 },
  { q: '最想对你说的一句话？', options: ['我爱你', '谢谢你', '有你真好', '三句都想说'], answer: 3 },
  { q: '如果可以重来，我会？', options: ['更早遇见你', '更勇敢表白', '更珍惜每一天', '不改变任何事'], answer: 3 },
  { q: '我们的爱情像什么？', options: ['春天的花', '夏天的风', '秋天的月', '冬天的暖阳'], answer: 3 },
  { q: '下辈子还选你吗？', options: ['当然', '必须的', '想都不用想', '每辈子都选你'], answer: 3 },
]

const mode = ref('menu')
const questions = ref([])
const currentQ = ref(0)
const answered = ref(false)
const selectedIdx = ref(-1)
const scores = ref(0)
const timer = ref(10)
const streak = ref(0)
const bestStreak = ref(0)
let timerInterval = null

// 特效状态
const particles = ref([])
let pId = 0
const comboText = ref('')

function shuffle(arr) {
  const a = [...arr]
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]]
  }
  return a
}

function startQuiz(count) {
  questions.value = shuffle(allQuestions).slice(0, count)
  currentQ.value = 0
  answered.value = false
  selectedIdx.value = -1
  scores.value = 0
  streak.value = 0
  bestStreak.value = 0
  mode.value = 'playing'
  startTimer()
  animateInQuestion()
}

function startTimer() {
  timer.value = 10
  clearInterval(timerInterval)
  timerInterval = setInterval(() => {
    timer.value -= 0.05 // 更平滑的进度条更新 (50ms)
    if (timer.value <= 0) {
      clearInterval(timerInterval)
      timer.value = 0
      selectAnswer(-1, null) // 超时
    }
  }, 50)
}

function spawnParticles(x, y, isCorrect) {
  for (let i = 0; i < (isCorrect ? 15 : 5); i++) {
    const id = pId++
    particles.value.push({
      id,
      x, y,
      tx: (Math.random() - 0.5) * 200,
      ty: (Math.random() - 0.5) * 200 - 50,
      size: Math.random() * 15 + 10,
      emoji: isCorrect ? ['❤️', '💖', '✨'][Math.floor(Math.random()*3)] : '💧'
    })
    setTimeout(() => {
      particles.value = particles.value.filter(p => p.id !== id)
    }, 1000)
  }
}

function showCombo(amount) {
  comboText.value = `COMBO x${amount}!`
  nextTick(() => {
    gsap.fromTo('.combo-popup',
        { scale: 0.5, opacity: 0, y: 20 },
        { scale: 1.2, opacity: 1, y: -20, duration: 0.4, ease: 'back.out(2)', yoyo: true, repeat: 1, onComplete: () => { comboText.value = '' } }
    )
  })
}

function animateInQuestion() {
  nextTick(() => {
    gsap.fromTo('.q-text', { opacity: 0, y: 20 }, { opacity: 1, y: 0, duration: 0.5, ease: 'power2.out' })
    gsap.fromTo('.option-btn',
        { opacity: 0, x: -30 },
        { opacity: 1, x: 0, duration: 0.4, stagger: 0.1, ease: 'back.out(1.2)' }
    )
  })
}

function selectAnswer(idx, event) {
  if (answered.value) return
  clearInterval(timerInterval)
  selectedIdx.value = idx
  answered.value = true

  const correct = questions.value[currentQ.value].answer
  const isCorrect = idx === correct

  // 触发粒子
  if (event) {
    const rect = event.currentTarget.getBoundingClientRect()
    spawnParticles(rect.left + rect.width / 2, rect.top + rect.height / 2, isCorrect)
  } else {
    spawnParticles(window.innerWidth / 2, window.innerHeight / 2, false)
  }

  // 弹窗震动与算分
  if (isCorrect) {
    const timeBonus = Math.floor(Math.max(0, timer.value) * 2)
    const streakBonus = streak.value >= 2 ? streak.value * 5 : 0
    scores.value += 10 + timeBonus + streakBonus
    streak.value++
    if (streak.value > bestStreak.value) bestStreak.value = streak.value

    if (streak.value >= 2) showCombo(streak.value)
  } else {
    streak.value = 0
    gsap.fromTo('.quiz-card', { x: -10 }, { x: 10, duration: 0.1, yoyo: true, repeat: 3, clearProps: 'x' })
  }

  // 延迟进入下一题
  setTimeout(() => {
    if (currentQ.value < questions.value.length - 1) {
      currentQ.value++
      answered.value = false
      selectedIdx.value = -1
      startTimer()
      animateInQuestion()
    } else {
      mode.value = 'result'
      clearInterval(timerInterval)
      nextTick(() => {
        gsap.from('.result-card', { scale: 0.8, opacity: 0, y: 50, duration: 0.8, ease: 'elastic.out(1, 0.6)' })
        gsap.from('.stat', { opacity: 0, y: 20, duration: 0.5, stagger: 0.2, delay: 0.3 })
      })
    }
  }, 1200)
}

const resultTitle = computed(() => {
  const pct = scores.value / (questions.value.length * 30) // 大致满分评估
  if (pct >= 0.8) return { emoji: '💯', title: '心有灵犀！', text: '简直是我肚子里的蛔虫，太懂我了！' }
  if (pct >= 0.5) return { emoji: '💕', title: '默契不错！', text: '我们越来越懂彼此了哦。' }
  return { emoji: '🥰', title: '没关系~', text: '不管选什么，最后的答案都是你。' }
})
</script>

<template>
  <div class="quiz-game">
    <div class="glow-orb orb-1"></div>
    <div class="glow-orb orb-2"></div>

    <div class="particles-container">
      <div
          v-for="p in particles"
          :key="p.id"
          class="particle"
          :style="{ left: p.x + 'px', top: p.y + 'px', '--tx': p.tx + 'px', '--ty': p.ty + 'px', fontSize: p.size + 'px' }"
      >{{ p.emoji }}</div>
    </div>

    <div v-if="comboText" class="combo-popup">{{ comboText }}</div>

    <h2 class="section-title" v-if="mode !== 'playing'">默契大考验</h2>

    <div v-if="mode === 'menu'" class="menu glass-panel">
      <p class="section-sub">测测我们的心有多近</p>
      <div class="mode-cards">
        <div class="mode-card" @click="startQuiz(5)">
          <div class="card-glare"></div>
          <span class="mode-emoji">⚡</span>
          <span class="mode-name">心跳快问</span>
          <span class="mode-desc">5题 · 争分夺秒</span>
        </div>
        <div class="mode-card" @click="startQuiz(10)">
          <div class="card-glare"></div>
          <span class="mode-emoji">🎯</span>
          <span class="mode-name">灵魂伴侣</span>
          <span class="mode-desc">10题 · 深度考验</span>
        </div>
      </div>
      <p class="menu-tip">✨ 答对得分，速度越快分越高，连续答对有高额爆分加成！</p>
    </div>

    <template v-if="mode === 'playing'">
      <div class="timer-bar-container" :class="{ danger: timer <= 3 }">
        <div class="timer-bar-fill" :style="{ width: (timer / 10) * 100 + '%' }"></div>
      </div>

      <div class="game-hud">
        <div class="hud-item score">
          <span class="hud-icon">💎</span>
          <span>{{ scores }}</span>
        </div>
        <div class="hud-item progress-text">
          <span>第 {{ currentQ + 1 }} / {{ questions.length }} 题</span>
        </div>
        <div class="hud-item streak" :class="{ active: streak >= 2 }">
          <span class="hud-icon">🔥</span>
          <span>{{ streak }}</span>
        </div>
      </div>

      <div class="quiz-card glass-panel">
        <h3 class="q-text">{{ questions[currentQ].q }}</h3>

        <div class="options">
          <button
              v-for="(opt, i) in questions[currentQ].options"
              :key="i"
              class="option-btn"
              :class="{
              selected: selectedIdx === i,
              correct: answered && i === questions[currentQ].answer,
              wrong: answered && selectedIdx === i && i !== questions[currentQ].answer
            }"
              :disabled="answered"
              @click="selectAnswer(i, $event)"
          >
            <div class="opt-content">
              <span class="opt-letter">{{ ['A', 'B', 'C', 'D'][i] }}</span>
              <span class="opt-text">{{ opt }}</span>
            </div>
            <span v-if="answered && i === questions[currentQ].answer" class="opt-feedback correct-mark">✓</span>
            <span v-if="answered && selectedIdx === i && i !== questions[currentQ].answer" class="opt-feedback wrong-mark">✗</span>
          </button>
        </div>
      </div>
    </template>

    <div v-if="mode === 'result'" class="result-card glass-panel">
      <div class="result-header">
        <div class="result-emoji">{{ resultTitle.emoji }}</div>
        <h3 class="result-title">{{ resultTitle.title }}</h3>
        <p class="result-text">{{ resultTitle.text }}</p>
      </div>

      <div class="result-stats">
        <div class="stat primary">
          <span class="stat-label">总得分</span>
          <span class="stat-val">{{ scores }}</span>
        </div>
        <div class="stat-divider"></div>
        <div class="stat">
          <span class="stat-label">最高连击</span>
          <span class="stat-val">{{ bestStreak }}</span>
        </div>
      </div>

      <div class="result-actions">
        <button class="game-btn restart-btn" @click="startQuiz(questions.length)">再测一次</button>
        <button class="game-btn back-btn" @click="mode = 'menu'">返回大厅</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.quiz-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 76px);
  padding: 40px 15px;
  position: relative;
  overflow: hidden;
  background: radial-gradient(circle at top, #1a1525, #0a0a10);
  font-family: 'PingFang SC', sans-serif;
}

/* 氛围光晕 */
.glow-orb { position: absolute; border-radius: 50%; filter: blur(90px); opacity: 0.15; pointer-events: none; z-index: 0; }
.orb-1 { width: 350px; height: 350px; background: #ff6b9d; top: -10%; left: -10%; animation: drift 10s ease-in-out infinite alternate; }
.orb-2 { width: 300px; height: 300px; background: #764ba2; bottom: 10%; right: -10%; animation: drift 12s ease-in-out infinite alternate-reverse; }
@keyframes drift { 0% { transform: translate(0, 0); } 100% { transform: translate(50px, 30px); } }

.glass-panel {
  background: rgba(255, 255, 255, 0.04);
  backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 24px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  position: relative;
  z-index: 2;
}

.section-title { font-size: 1.8rem; font-weight: 800; margin-bottom: 5px; background: linear-gradient(135deg, #fff, #f0c4ff); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.section-sub { font-size: 0.9rem; color: rgba(255, 255, 255, 0.5); margin-bottom: 30px; }

/* 粒子系统 */
.particles-container { position: fixed; inset: 0; pointer-events: none; z-index: 100; overflow: hidden; }
.particle {
  position: absolute; pointer-events: none; opacity: 0;
  animation: burst 1s cubic-bezier(0.1, 0.8, 0.3, 1) forwards;
  filter: drop-shadow(0 2px 5px rgba(255, 107, 157, 0.5));
  transform: translate(-50%, -50%); /* 居中鼠标点击点 */
}
@keyframes burst {
  0% { transform: translate(-50%, -50%) scale(0); opacity: 1; }
  100% { transform: translate(calc(-50% + var(--tx)), calc(-50% + var(--ty))) scale(1.5) rotate(45deg); opacity: 0; }
}

/* COMBO 弹窗 */
.combo-popup {
  position: fixed; top: 30%; left: 50%; transform: translateX(-50%);
  font-size: 3rem; font-weight: 900; font-style: italic; z-index: 100; pointer-events: none;
  background: linear-gradient(135deg, #ffd700, #ff5722); -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 5px 15px rgba(255, 87, 34, 0.6));
}

/* --- 菜单区 --- */
.menu { padding: 40px 20px; max-width: 450px; width: 100%; text-align: center; }
.mode-cards { display: flex; gap: 15px; flex-wrap: wrap; justify-content: center; margin-bottom: 25px; }
.mode-card {
  flex: 1; min-width: 140px; padding: 30px 20px; border-radius: 16px; cursor: pointer;
  background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.06);
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1); position: relative; overflow: hidden;
}
.card-glare { position: absolute; inset: 0; background: linear-gradient(135deg, rgba(255,255,255,0.1) 0%, transparent 50%); opacity: 0; transition: opacity 0.3s; }
.mode-card:hover { transform: translateY(-5px); background: rgba(255, 107, 157, 0.08); border-color: rgba(255, 107, 157, 0.3); box-shadow: 0 10px 25px rgba(255, 107, 157, 0.2); }
.mode-card:hover .card-glare { opacity: 1; }
.mode-emoji { font-size: 38px; display: block; margin-bottom: 12px; filter: drop-shadow(0 2px 8px rgba(0,0,0,0.3)); }
.mode-name { display: block; font-size: 1.1rem; font-weight: 700; color: #fff; margin-bottom: 6px; }
.mode-desc { display: block; font-size: 0.75rem; color: rgba(255, 255, 255, 0.5); }
.menu-tip { font-size: 0.8rem; color: #ffb6c1; opacity: 0.8; line-height: 1.5; }

/* --- 游戏区 --- */
.timer-bar-container { position: fixed; top: 0; left: 0; width: 100%; height: 6px; background: rgba(0,0,0,0.3); z-index: 50; }
.timer-bar-fill { height: 100%; background: linear-gradient(90deg, #00f2fe, #4facfe); transition: width 0.05s linear; box-shadow: 0 0 15px rgba(79, 172, 254, 0.6); }
.timer-bar-container.danger .timer-bar-fill { background: linear-gradient(90deg, #ff0844, #ffb199); box-shadow: 0 0 20px rgba(255, 8, 68, 0.8); }
.timer-bar-container.danger { animation: screenRedFlash 0.5s infinite; }
@keyframes screenRedFlash { 0%, 100% { box-shadow: 0 0 0 transparent; } 50% { box-shadow: 0 10px 50px rgba(255, 0, 0, 0.2) inset; } }

.game-hud { display: flex; justify-content: space-between; align-items: center; width: 100%; max-width: 450px; margin-bottom: 20px; z-index: 2; }
.hud-item { display: flex; align-items: center; gap: 6px; background: rgba(0,0,0,0.4); padding: 8px 16px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.05); font-weight: 600; font-size: 0.95rem; }
.hud-icon { font-size: 1.1rem; }
.score { color: #4facfe; }
.streak { color: rgba(255,255,255,0.3); transition: all 0.3s; }
.streak.active { color: #ff9800; border-color: rgba(255, 152, 0, 0.4); box-shadow: 0 0 15px rgba(255, 152, 0, 0.2); }
.progress-text { background: transparent; border: none; color: rgba(255,255,255,0.5); font-size: 0.85rem; }

.quiz-card { padding: 40px 25px; width: 100%; max-width: 450px; }
.q-text { font-size: 1.35rem; color: #fff; margin-bottom: 30px; font-weight: 600; line-height: 1.5; text-align: center; text-shadow: 0 2px 4px rgba(0,0,0,0.5); }

.options { display: flex; flex-direction: column; gap: 12px; }
.option-btn {
  display: flex; justify-content: space-between; align-items: center;
  padding: 16px 20px; border-radius: 16px; background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.8); cursor: pointer; transition: all 0.2s; overflow: hidden; position: relative;
}
.opt-content { display: flex; align-items: center; gap: 15px; }
.opt-letter { width: 30px; height: 30px; display: flex; align-items: center; justify-content: center; border-radius: 8px; background: rgba(255, 255, 255, 0.1); font-weight: 700; font-size: 0.9rem; }
.opt-text { font-size: 1rem; font-weight: 500; text-align: left; }

.option-btn:hover:not(:disabled) { background: rgba(255, 255, 255, 0.08); transform: scale(1.02); }
.option-btn.selected { transform: scale(0.98); opacity: 0.8; }

/* 结算状态高亮 */
.option-btn.correct { background: rgba(76, 175, 80, 0.2); border-color: #4caf50; color: #fff; box-shadow: 0 0 20px rgba(76, 175, 80, 0.3); }
.option-btn.correct .opt-letter { background: #4caf50; color: #fff; }
.option-btn.wrong { background: rgba(244, 67, 54, 0.15); border-color: rgba(244, 67, 54, 0.4); opacity: 0.6; }
.option-btn.wrong .opt-letter { background: rgba(244, 67, 54, 0.5); }

.opt-feedback { font-size: 1.4rem; font-weight: 900; animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.correct-mark { color: #4caf50; text-shadow: 0 0 10px #4caf50; }
.wrong-mark { color: #f44336; }
@keyframes popIn { 0% { transform: scale(0); } 100% { transform: scale(1); } }

/* --- 结算区 --- */
.result-card { padding: 50px 30px; width: 100%; max-width: 400px; text-align: center; }
.result-header { margin-bottom: 35px; }
.result-emoji { font-size: 64px; margin-bottom: 15px; filter: drop-shadow(0 10px 15px rgba(0,0,0,0.3)); animation: float 3s ease-in-out infinite; }
@keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
.result-title { font-size: 1.8rem; font-weight: 800; color: #fff; margin-bottom: 10px; }
.result-text { font-size: 1rem; color: #ffb6c1; line-height: 1.5; }

.result-stats { display: flex; align-items: center; justify-content: center; background: rgba(0,0,0,0.3); border-radius: 20px; padding: 20px; margin-bottom: 35px; border: 1px solid rgba(255,255,255,0.05); }
.stat { display: flex; flex-direction: column; gap: 5px; flex: 1; }
.stat-divider { width: 1px; height: 40px; background: rgba(255,255,255,0.1); }
.stat-label { font-size: 0.8rem; color: rgba(255,255,255,0.5); }
.stat-val { font-size: 1.8rem; font-weight: 800; color: #fff; font-variant-numeric: tabular-nums; }
.primary .stat-val { color: #ff6b9d; text-shadow: 0 0 15px rgba(255, 107, 157, 0.4); font-size: 2.2rem; }

.result-actions { display: flex; gap: 15px; justify-content: center; }
.game-btn { padding: 14px 28px; border-radius: 30px; font-weight: 600; font-size: 1rem; cursor: pointer; transition: all 0.3s; border: none; }
.restart-btn { background: linear-gradient(135deg, #ff6b9d, #c44dff); color: #fff; box-shadow: 0 5px 20px rgba(196, 77, 255, 0.3); }
.restart-btn:hover { transform: translateY(-3px); box-shadow: 0 8px 25px rgba(196, 77, 255, 0.5); }
.back-btn { background: rgba(255,255,255,0.08); color: #fff; border: 1px solid rgba(255,255,255,0.15); }
.back-btn:hover { background: rgba(255,255,255,0.15); transform: translateY(-3px); }
</style>