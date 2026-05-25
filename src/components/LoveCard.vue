<script setup>
import { ref } from 'vue'
import gsap from 'gsap'

const cards = [
  { type: '夸夸卡', text: '你今天也超级可爱。', emoji: '🥰', color: '#ff6b9d' },
  { type: '约会卡', text: '今天一起吃点好吃的。', emoji: '🍜', color: '#ffa726' },
  { type: '抱抱卡', text: '欠你一个很久很久的抱抱。', emoji: '🤗', color: '#ab47bc' },
  { type: '想念卡', text: '我现在有点想你。', emoji: '💭', color: '#42a5f5' },
  { type: '亲亲卡', text: '想亲亲你的额头。', emoji: '😘', color: '#ec407a' },
  { type: '撒娇卡', text: '今天可以多陪我一会儿吗？', emoji: '🥺', color: '#7e57c2' },
  { type: '表白卡', text: '我今天也很喜欢你。', emoji: '💗', color: '#e91e63' },
  { type: '未来卡', text: '想和你一起变老。', emoji: '🌅', color: '#26a69a' },
]

const currentCard = ref(null)
const isDrawing = ref(false)

function draw() {
  if (isDrawing.value) return
  isDrawing.value = true
  currentCard.value = null

  setTimeout(() => {
    const idx = Math.floor(Math.random() * cards.length)
    currentCard.value = cards[idx]
    isDrawing.value = false

    setTimeout(() => {
      gsap.from('.drawn-card', { rotateY: 180, opacity: 0, duration: 0.6, ease: 'back.out(1.5)' })
    }, 10)
  }, 600)
}
</script>

<template>
  <div class="love-card">
    <h2 class="section-title">今日恋爱抽卡</h2>
    <p class="card-desc">每天一张专属卡片，看看今天的甜蜜是什么</p>

    <button class="draw-btn" :class="{ drawing: isDrawing }" @click="draw">
      <span v-if="!isDrawing">✨ 抽一张卡</span>
      <span v-else>抽取中...</span>
    </button>

    <div v-if="currentCard" class="drawn-card" :style="{ borderColor: currentCard.color + '40' }">
      <div class="card-emoji">{{ currentCard.emoji }}</div>
      <div class="card-type" :style="{ color: currentCard.color }">{{ currentCard.type }}</div>
      <div class="card-text">{{ currentCard.text }}</div>
    </div>

    <div v-else class="card-placeholder">
      <div class="placeholder-icon">🎴</div>
      <p>点击上方按钮抽取今日卡片</p>
    </div>
  </div>
</template>

<style scoped>
.love-card {
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

.card-desc {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.5);
  margin-bottom: 32px;
}

.draw-btn {
  padding: 14px 36px;
  border-radius: 30px;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  color: #fff;
  font-size: 1rem;
  transition: all 0.3s;
  box-shadow: 0 4px 20px rgba(255, 107, 157, 0.3);
  margin-bottom: 40px;
}

.draw-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 30px rgba(255, 107, 157, 0.5);
}

.draw-btn.drawing {
  opacity: 0.7;
  animation: shake 0.5s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: rotate(0); }
  25% { transform: rotate(-5deg); }
  75% { transform: rotate(5deg); }
}

.drawn-card {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid;
  border-radius: 20px;
  padding: 40px;
  text-align: center;
  max-width: 300px;
  width: 100%;
  backdrop-filter: blur(10px);
}

.card-emoji {
  font-size: 48px;
  margin-bottom: 16px;
}

.card-type {
  font-size: 1.1rem;
  font-weight: 600;
  margin-bottom: 12px;
}

.card-text {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.8);
  line-height: 1.6;
}

.card-placeholder {
  text-align: center;
  color: rgba(255, 255, 255, 0.3);
}

.placeholder-icon {
  font-size: 48px;
  margin-bottom: 12px;
  opacity: 0.5;
}

.card-placeholder p {
  font-size: 0.85rem;
}
</style>
