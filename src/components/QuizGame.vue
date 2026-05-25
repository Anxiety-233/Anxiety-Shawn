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
  }
]

const currentQ = ref(0)
const answered = ref(false)
const showResult = ref(false)
const answers = ref([])

function selectAnswer(idx) {
  answers.value.push(idx)
  answered.value = true

  setTimeout(() => {
    if (currentQ.value < questions.length - 1) {
      currentQ.value++
      answered.value = false
      gsap.from('.quiz-question', { opacity: 0, x: 30, duration: 0.4 })
    } else {
      showResult.value = true
      setTimeout(() => {
        gsap.from('.result-card', { scale: 0.8, opacity: 0, duration: 0.6, ease: 'back.out(1.7)' })
      }, 10)
    }
  }, 500)
}

function restart() {
  currentQ.value = 0
  answered.value = false
  showResult.value = false
  answers.value = []
}
</script>

<template>
  <div class="quiz-game">
    <h2 class="section-title">默契小游戏</h2>

    <template v-if="!showResult">
      <div class="progress">
        <div class="progress-bar" :style="{ width: ((currentQ + 1) / questions.length * 100) + '%' }"></div>
      </div>
      <div class="quiz-question">
        <p class="q-number">第 {{ currentQ + 1 }} / {{ questions.length }} 题</p>
        <h3 class="q-text">{{ questions[currentQ].q }}</h3>
        <div class="options">
          <button
            v-for="(opt, i) in questions[currentQ].options"
            :key="i"
            class="option-btn"
            :class="{ selected: answered && answers[currentQ] === i }"
            :disabled="answered"
            @click="selectAnswer(i)"
          >
            <span class="opt-letter">{{ ['A', 'B', 'C', 'D'][i] }}</span>
            {{ opt }}
          </button>
        </div>
      </div>
    </template>

    <div v-else class="result-card">
      <div class="result-emoji">💯</div>
      <h3 class="result-title">结果：满分。</h3>
      <p class="result-text">因为答案本来就是你。</p>
      <button class="restart-btn" @click="restart">再玩一次</button>
    </div>
  </div>
</template>

<style scoped>
.quiz-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 70px);
  padding: 40px 20px;
}

.section-title {
  font-size: 1.5rem;
  margin-bottom: 24px;
  color: rgba(255, 255, 255, 0.9);
}

.progress {
  width: 100%;
  max-width: 400px;
  height: 4px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 2px;
  margin-bottom: 40px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #ff6b9d, #c44dff);
  border-radius: 2px;
  transition: width 0.4s ease;
}

.quiz-question {
  max-width: 400px;
  width: 100%;
  text-align: center;
}

.q-number {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.4);
  margin-bottom: 16px;
}

.q-text {
  font-size: 1.2rem;
  margin-bottom: 32px;
  color: rgba(255, 255, 255, 0.9);
  font-weight: 500;
}

.options {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.option-btn {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 20px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
  text-align: left;
  transition: all 0.3s;
}

.option-btn:hover:not(:disabled) {
  background: rgba(255, 107, 157, 0.1);
  border-color: rgba(255, 107, 157, 0.3);
}

.option-btn.selected {
  background: rgba(255, 107, 157, 0.2);
  border-color: rgba(255, 107, 157, 0.5);
  color: #fff;
}

.opt-letter {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  font-size: 0.8rem;
  flex-shrink: 0;
}

.result-card {
  text-align: center;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 107, 157, 0.2);
  border-radius: 24px;
  padding: 50px 40px;
  max-width: 320px;
}

.result-emoji {
  font-size: 56px;
  margin-bottom: 20px;
}

.result-title {
  font-size: 1.4rem;
  margin-bottom: 12px;
  color: #ff6b9d;
}

.result-text {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 30px;
}

.restart-btn {
  padding: 10px 28px;
  border-radius: 20px;
  background: rgba(255, 107, 157, 0.2);
  color: #ff6b9d;
  font-size: 0.9rem;
  transition: all 0.3s;
}

.restart-btn:hover {
  background: rgba(255, 107, 157, 0.3);
}
</style>
