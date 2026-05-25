<script setup>
import { ref, computed } from 'vue'
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
}

function startTimer() {
  timer.value = 10
  clearInterval(timerInterval)
  timerInterval = setInterval(() => {
    timer.value--
    if (timer.value <= 0) {
      clearInterval(timerInterval)
      selectAnswer(-1)
    }
  }, 1000)
}

function selectAnswer(idx) {
  if (answered.value) return
  clearInterval(timerInterval)
  selectedIdx.value = idx
  answered.value = true

  const correct = questions.value[currentQ.value].answer
  const isCorrect = idx === correct
  if (isCorrect) {
    const timeBonus = Math.max(0, timer.value)
    const streakBonus = streak.value >= 3 ? streak.value : 0
    scores.value += 10 + timeBonus + streakBonus
    streak.value++
    if (streak.value > bestStreak.value) bestStreak.value = streak.value
  } else {
    streak.value = 0
  }

  setTimeout(() => {
    if (currentQ.value < questions.value.length - 1) {
      currentQ.value++
      answered.value = false
      selectedIdx.value = -1
      startTimer()
      gsap.from('.quiz-question', { opacity: 0, x: 40, duration: 0.4, ease: 'power3.out' })
    } else {
      mode.value = 'result'
      clearInterval(timerInterval)
      setTimeout(() => {
        gsap.from('.result-card', { scale: 0.7, opacity: 0, duration: 0.8, ease: 'back.out(1.7)' })
      }, 10)
    }
  }, 800)
}

const resultTitle = computed(() => {
  const pct = scores.value / (questions.value.length * 20)
  if (pct >= 0.8) return { emoji: '💯', title: '心有灵犀！', text: '你完全懂我的心。' }
  if (pct >= 0.5) return { emoji: '💕', title: '默契不错！', text: '我们越来越懂彼此了。' }
  return { emoji: '🥰', title: '没关系~', text: '不管选什么，答案都是你。' }
})
</script>

<template>
  <div class="quiz-game">
    <h2 class="section-title">默契小游戏</h2>

    <div v-if="mode === 'menu'" class="menu">
      <p class="section-sub">测测我们的心有多近</p>
      <div class="mode-cards">
        <div class="mode-card" @click="startQuiz(5)">
          <span class="mode-emoji">⚡</span>
          <span class="mode-name">快速模式</span>
          <span class="mode-desc">5题 · 限时10秒</span>
        </div>
        <div class="mode-card" @click="startQuiz(10)">
          <span class="mode-emoji">🎯</span>
          <span class="mode-name">完整模式</span>
          <span class="mode-desc">10题 · 限时10秒</span>
        </div>
      </div>
      <p class="menu-tip">答对得分，速度越快分越高，连续答对有连击加成！</p>
    </div>

    <template v-if="mode === 'playing'">
      <div class="game-hud">
        <span class="hud-score">{{ scores }}分</span>
        <span class="hud-timer" :class="{ urgent: timer <= 3 }">{{ timer }}s</span>
        <span v-if="streak >= 3" class="hud-streak">🔥x{{ streak }}</span>
      </div>

      <div class="progress">
        <div class="progress-bar" :style="{ width: ((currentQ + 1) / questions.length * 100) + '%' }"></div>
      </div>

      <div class="quiz-question">
        <p class="q-number">{{ currentQ + 1 }} / {{ questions.length }}</p>
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
            @click="selectAnswer(i)"
          >
            <span class="opt-letter">{{ ['A', 'B', 'C', 'D'][i] }}</span>
            <span class="opt-text">{{ opt }}</span>
            <span v-if="answered && i === questions[currentQ].answer" class="opt-check">✓</span>
          </button>
        </div>
      </div>
    </template>

    <div v-if="mode === 'result'" class="result-card">
      <div class="result-emoji">{{ resultTitle.emoji }}</div>
      <h3 class="result-title">{{ resultTitle.title }}</h3>
      <p class="result-text">{{ resultTitle.text }}</p>
      <div class="result-stats">
        <div class="stat">
          <span class="stat-val">{{ scores }}</span>
          <span class="stat-label">总分</span>
        </div>
        <div class="stat">
          <span class="stat-val">{{ bestStreak }}</span>
          <span class="stat-label">最长连击</span>
        </div>
      </div>
      <div class="result-actions">
        <button class="restart-btn" @click="startQuiz(questions.length)">再玩一次</button>
        <button class="back-btn" @click="mode = 'menu'">返回</button>
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
  padding: 40px 20px;
}

.section-title {
  font-size: 1.6rem;
  font-weight: 600;
  margin-bottom: 16px;
  background: linear-gradient(135deg, #fff, #f0c4ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.section-sub {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.4);
  margin-bottom: 32px;
}

.menu { text-align: center; }

.mode-cards {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  justify-content: center;
  margin-bottom: 20px;
}

.mode-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 28px 32px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  cursor: pointer;
  transition: all 0.3s;
  min-width: 150px;
}

.mode-card:hover {
  background: rgba(255, 107, 157, 0.08);
  border-color: rgba(255, 107, 157, 0.25);
  transform: translateY(-4px);
}

.mode-emoji { font-size: 32px; }
.mode-name { font-size: 1rem; font-weight: 500; color: rgba(255, 255, 255, 0.9); }
.mode-desc { font-size: 0.75rem; color: rgba(255, 255, 255, 0.4); }
.menu-tip { font-size: 0.75rem; color: rgba(255, 255, 255, 0.3); }

.game-hud {
  display: flex;
  gap: 16px;
  align-items: center;
  margin-bottom: 16px;
  font-size: 0.85rem;
}

.hud-score { color: #ff6b9d; font-weight: 600; }
.hud-timer { color: rgba(255, 255, 255, 0.6); font-variant-numeric: tabular-nums; }
.hud-timer.urgent { color: #ff4444; animation: blink 0.5s infinite; }
.hud-streak { color: #ffd700; font-weight: 600; }

@keyframes blink { 50% { opacity: 0.5; } }

.progress {
  width: 100%;
  max-width: 400px;
  height: 3px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 2px;
  margin-bottom: 32px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #ff6b9d, #c44dff);
  border-radius: 2px;
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.quiz-question { max-width: 420px; width: 100%; text-align: center; }
.q-number { font-size: 0.75rem; color: rgba(255, 255, 255, 0.3); margin-bottom: 16px; letter-spacing: 2px; }
.q-text { font-size: 1.3rem; margin-bottom: 32px; color: rgba(255, 255, 255, 0.9); font-weight: 500; line-height: 1.5; }

.options { display: flex; flex-direction: column; gap: 10px; }

.option-btn {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 15px 18px;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.75);
  font-size: 0.9rem;
  text-align: left;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.option-btn:hover:not(:disabled) { background: rgba(255, 107, 157, 0.08); border-color: rgba(255, 107, 157, 0.25); transform: translateX(4px); }
.option-btn.selected { background: linear-gradient(135deg, rgba(255, 107, 157, 0.15), rgba(196, 77, 255, 0.1)); border-color: rgba(255, 107, 157, 0.4); color: #fff; }
.option-btn.correct { background: rgba(76, 175, 80, 0.15); border-color: rgba(76, 175, 80, 0.5); color: #4caf50; }
.option-btn.wrong { background: rgba(244, 67, 54, 0.1); border-color: rgba(244, 67, 54, 0.3); color: rgba(255, 255, 255, 0.5); }

.opt-letter { width: 28px; height: 28px; display: flex; align-items: center; justify-content: center; border-radius: 8px; background: rgba(255, 255, 255, 0.06); font-size: 0.8rem; font-weight: 600; flex-shrink: 0; }
.opt-text { flex: 1; }
.opt-check { color: #4caf50; font-weight: 600; }

.result-card {
  text-align: center;
  background: linear-gradient(145deg, rgba(255, 107, 157, 0.06), rgba(196, 77, 255, 0.04));
  border: 1px solid rgba(255, 107, 157, 0.2);
  border-radius: 28px;
  padding: 48px 40px;
  max-width: 360px;
  width: 100%;
}

.result-emoji { font-size: 56px; margin-bottom: 16px; }
.result-title { font-size: 1.5rem; margin-bottom: 12px; background: linear-gradient(135deg, #ff6b9d, #c44dff); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
.result-text { font-size: 0.95rem; color: rgba(255, 255, 255, 0.6); margin-bottom: 24px; }

.result-stats { display: flex; gap: 32px; justify-content: center; margin-bottom: 28px; }
.stat { text-align: center; }
.stat-val { display: block; font-size: 1.8rem; font-weight: 700; color: #ff6b9d; }
.stat-label { font-size: 0.75rem; color: rgba(255, 255, 255, 0.4); }

.result-actions { display: flex; gap: 12px; justify-content: center; }
.restart-btn { padding: 10px 24px; border-radius: 20px; background: rgba(255, 107, 157, 0.2); border: 1px solid rgba(255, 107, 157, 0.3); color: #ff6b9d; font-size: 0.85rem; transition: all 0.3s; }
.restart-btn:hover { background: rgba(255, 107, 157, 0.3); }
.back-btn { padding: 10px 24px; border-radius: 20px; background: rgba(255, 255, 255, 0.06); border: 1px solid rgba(255, 255, 255, 0.1); color: rgba(255, 255, 255, 0.6); font-size: 0.85rem; transition: all 0.3s; }
.back-btn:hover { background: rgba(255, 255, 255, 0.1); }
</style>
