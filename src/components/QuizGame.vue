<script setup>
import { ref } from 'vue'
import gsap from 'gsap'

const questions = [
  {
    q: '我最想和你一起做什么？',
    options: ['吃饭', '散步', '看电影', '什么都不做，只待在一起']
  },
  {
    q: '如果可以送你一样东西？',
    options: ['星星', '一整天的陪伴', '一个永远的拥抱', '全世界']
  },
  {
    q: '最喜欢什么时候的你？',
    options: ['笑的时候', '发呆的时候', '撒娇的时候', '每时每刻']
  },
  {
    q: '我们最像哪对CP？',
    options: ['甜蜜日常型', '搞笑吵闹型', '安静陪伴型', '独一无二型']
  },
  {
    q: '如果世界末日，我会？',
    options: ['带你逃跑', '抱紧你', '和你一起看最后的星空', '以上全部']
  }
]

const currentQ = ref(0)
const answered = ref(false)
const showResult = ref(false)
const answers = ref([])
const selectedIdx = ref(-1)

function selectAnswer(idx) {
  if (answered.value) return
  selectedIdx.value = idx
  answers.value.push(idx)
  answered.value = true

  setTimeout(() => {
    if (currentQ.value < questions.length - 1) {
      currentQ.value++
      answered.value = false
      selectedIdx.value = -1
      gsap.from('.quiz-question', { opacity: 0, x: 40, duration: 0.5, ease: 'power3.out' })
    } else {
      showResult.value = true
      setTimeout(() => {
        gsap.from('.result-card', { scale: 0.7, opacity: 0, duration: 0.8, ease: 'back.out(1.7)' })
        gsap.from('.confetti', { opacity: 0, y: -20, stagger: 0.05, duration: 0.4, delay: 0.3 })
      }, 10)
    }
  }, 600)
}

function restart() {
  currentQ.value = 0
  answered.value = false
  showResult.value = false
  answers.value = []
  selectedIdx.value = -1
}
</script>

<template>
  <div class="quiz-game">
    <h2 class="section-title">默契小游戏</h2>
    <p class="section-sub">测测我们的心有多近</p>

    <template v-if="!showResult">
      <div class="progress">
        <div class="progress-bar" :style="{ width: ((currentQ + 1) / questions.length * 100) + '%' }"></div>
        <div class="progress-dots">
          <span
            v-for="i in questions.length"
            :key="i"
            class="dot"
            :class="{ done: i - 1 < currentQ, current: i - 1 === currentQ }"
          ></span>
        </div>
      </div>

      <div class="quiz-question">
        <p class="q-number">{{ currentQ + 1 }} / {{ questions.length }}</p>
        <h3 class="q-text">{{ questions[currentQ].q }}</h3>
        <div class="options">
          <button
            v-for="(opt, i) in questions[currentQ].options"
            :key="i"
            class="option-btn"
            :class="{ selected: selectedIdx === i }"
            :disabled="answered"
            @click="selectAnswer(i)"
          >
            <span class="opt-letter">{{ ['A', 'B', 'C', 'D'][i] }}</span>
            <span class="opt-text">{{ opt }}</span>
            <span v-if="selectedIdx === i" class="opt-check">✓</span>
          </button>
        </div>
      </div>
    </template>

    <div v-else class="result-card">
      <div class="confetti-container">
        <span v-for="i in 12" :key="i" class="confetti" :style="{ left: (i * 8) + '%', animationDelay: (i * 0.1) + 's' }">✦</span>
      </div>
      <div class="result-emoji">💯</div>
      <h3 class="result-title">默契满分！</h3>
      <p class="result-text">因为答案本来就是你。<br>不管选什么，有你就是对的。</p>
      <div class="result-hearts">💕 💕 💕</div>
      <button class="restart-btn" @click="restart">再玩一次</button>
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
  margin-bottom: 8px;
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

.progress {
  width: 100%;
  max-width: 400px;
  margin-bottom: 40px;
}

.progress-bar {
  height: 3px;
  background: linear-gradient(90deg, #ff6b9d, #c44dff);
  border-radius: 2px;
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 0 8px rgba(255, 107, 157, 0.4);
}

.progress-dots {
  display: flex;
  justify-content: space-between;
  margin-top: 12px;
}

.dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  transition: all 0.3s;
}

.dot.done {
  background: #ff6b9d;
  box-shadow: 0 0 6px rgba(255, 107, 157, 0.5);
}

.dot.current {
  background: #c44dff;
  box-shadow: 0 0 8px rgba(196, 77, 255, 0.6);
  transform: scale(1.3);
}

.quiz-question {
  max-width: 420px;
  width: 100%;
  text-align: center;
}

.q-number {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.3);
  margin-bottom: 16px;
  letter-spacing: 2px;
}

.q-text {
  font-size: 1.3rem;
  margin-bottom: 36px;
  color: rgba(255, 255, 255, 0.9);
  font-weight: 500;
  line-height: 1.5;
}

.options {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.option-btn {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 16px 20px;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.75);
  font-size: 0.9rem;
  text-align: left;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.option-btn:hover:not(:disabled) {
  background: rgba(255, 107, 157, 0.08);
  border-color: rgba(255, 107, 157, 0.25);
  transform: translateX(4px);
}

.option-btn.selected {
  background: linear-gradient(135deg, rgba(255, 107, 157, 0.15), rgba(196, 77, 255, 0.1));
  border-color: rgba(255, 107, 157, 0.4);
  color: #fff;
  transform: translateX(4px);
}

.opt-letter {
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.06);
  font-size: 0.8rem;
  font-weight: 600;
  flex-shrink: 0;
}

.opt-text {
  flex: 1;
}

.opt-check {
  color: #ff6b9d;
  font-weight: 600;
}

.result-card {
  text-align: center;
  background: linear-gradient(145deg, rgba(255, 107, 157, 0.06), rgba(196, 77, 255, 0.04));
  border: 1px solid rgba(255, 107, 157, 0.2);
  border-radius: 28px;
  padding: 56px 44px;
  max-width: 340px;
  position: relative;
  overflow: hidden;
  box-shadow: 0 16px 48px rgba(0, 0, 0, 0.2);
}

.confetti-container {
  position: absolute;
  inset: 0;
  pointer-events: none;
  overflow: hidden;
}

.confetti {
  position: absolute;
  top: -10px;
  font-size: 14px;
  color: rgba(255, 107, 157, 0.6);
  animation: fall 3s ease-in-out infinite;
}

@keyframes fall {
  0% { transform: translateY(-10px) rotate(0); opacity: 1; }
  100% { transform: translateY(300px) rotate(360deg); opacity: 0; }
}

.result-emoji {
  font-size: 64px;
  margin-bottom: 20px;
}

.result-title {
  font-size: 1.6rem;
  margin-bottom: 16px;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.result-text {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 20px;
  line-height: 1.8;
}

.result-hearts {
  font-size: 1.2rem;
  margin-bottom: 28px;
  letter-spacing: 4px;
}

.restart-btn {
  padding: 12px 32px;
  border-radius: 24px;
  background: rgba(255, 107, 157, 0.15);
  border: 1px solid rgba(255, 107, 157, 0.3);
  color: #ff6b9d;
  font-size: 0.9rem;
  transition: all 0.3s;
}

.restart-btn:hover {
  background: rgba(255, 107, 157, 0.25);
  transform: translateY(-2px);
}
</style>
