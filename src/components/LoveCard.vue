<script setup>
import { ref } from 'vue'
import gsap from 'gsap'

const cards = [
  { type: '夸夸卡', text: '你今天也超级可爱。', emoji: '🥰', color: '#ff6b9d', bg: 'rgba(255, 107, 157, 0.08)' },
  { type: '约会卡', text: '今天一起吃点好吃的。', emoji: '🍜', color: '#ffa726', bg: 'rgba(255, 167, 38, 0.08)' },
  { type: '抱抱卡', text: '欠你一个很久很久的抱抱。', emoji: '🤗', color: '#ab47bc', bg: 'rgba(171, 71, 188, 0.08)' },
  { type: '想念卡', text: '我现在有点想你。', emoji: '💭', color: '#42a5f5', bg: 'rgba(66, 165, 245, 0.08)' },
  { type: '亲亲卡', text: '想亲亲你的额头。', emoji: '😘', color: '#ec407a', bg: 'rgba(236, 64, 122, 0.08)' },
  { type: '撒娇卡', text: '今天可以多陪我一会儿吗？', emoji: '🥺', color: '#7e57c2', bg: 'rgba(126, 87, 194, 0.08)' },
  { type: '表白卡', text: '我今天也很喜欢你。', emoji: '💗', color: '#e91e63', bg: 'rgba(233, 30, 99, 0.08)' },
  { type: '未来卡', text: '想和你一起变老。', emoji: '🌅', color: '#26a69a', bg: 'rgba(38, 166, 154, 0.08)' },
  { type: '勇气卡', text: '有你在，什么都不怕。', emoji: '💪', color: '#5c6bc0', bg: 'rgba(92, 107, 192, 0.08)' },
  { type: '晚安卡', text: '今晚做个有我的梦。', emoji: '🌙', color: '#8d6e63', bg: 'rgba(141, 110, 99, 0.08)' },
]

const currentCard = ref(null)
const isDrawing = ref(false)
const history = ref([])

function draw() {
  if (isDrawing.value) return
  isDrawing.value = true
  currentCard.value = null

  setTimeout(() => {
    const idx = Math.floor(Math.random() * cards.length)
    currentCard.value = cards[idx]
    history.value.unshift(cards[idx])
    if (history.value.length > 5) history.value.pop()
    isDrawing.value = false

    setTimeout(() => {
      gsap.from('.drawn-card', {
        rotateY: 180,
        opacity: 0,
        scale: 0.8,
        duration: 0.7,
        ease: 'back.out(1.5)'
      })
    }, 10)
  }, 800)
}
</script>

<template>
  <div class="love-card">
    <h2 class="section-title">今日恋爱抽卡</h2>
    <p class="card-desc">每天一张专属卡片，看看今天的甜蜜是什么</p>

    <button class="draw-btn" :class="{ drawing: isDrawing }" @click="draw">
      <span class="btn-sparkle">✨</span>
      <span v-if="!isDrawing">抽一张卡</span>
      <span v-else>抽取中...</span>
    </button>

    <div v-if="currentCard" class="drawn-card" :style="{ '--card-color': currentCard.color, background: currentCard.bg }">
      <div class="card-shine"></div>
      <div class="card-emoji">{{ currentCard.emoji }}</div>
      <div class="card-type">{{ currentCard.type }}</div>
      <div class="card-divider"></div>
      <div class="card-text">{{ currentCard.text }}</div>
    </div>

    <div v-else-if="!isDrawing" class="card-placeholder">
      <div class="placeholder-cards">
        <div class="ph-card"></div>
        <div class="ph-card"></div>
        <div class="ph-card"></div>
      </div>
      <p>点击上方按钮抽取今日卡片</p>
    </div>

    <div v-if="history.length > 1" class="history">
      <p class="history-title">最近抽到</p>
      <div class="history-items">
        <span v-for="(h, i) in history.slice(1)" :key="i" class="history-item">{{ h.emoji }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.love-card {
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

.card-desc {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.4);
  margin-bottom: 36px;
}

.draw-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 14px 40px;
  border-radius: 30px;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  color: #fff;
  font-size: 1rem;
  font-weight: 500;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 4px 24px rgba(255, 107, 157, 0.3);
  margin-bottom: 40px;
  position: relative;
  overflow: hidden;
}

.draw-btn::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, transparent, rgba(255, 255, 255, 0.1), transparent);
  transform: translateX(-100%);
  transition: transform 0.6s;
}

.draw-btn:hover::before {
  transform: translateX(100%);
}

.draw-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 36px rgba(255, 107, 157, 0.45);
}

.draw-btn.drawing {
  animation: shake 0.6s ease-in-out;
}

.btn-sparkle {
  font-size: 1.1rem;
}

@keyframes shake {
  0%, 100% { transform: rotate(0) scale(1); }
  20% { transform: rotate(-5deg) scale(1.02); }
  40% { transform: rotate(5deg) scale(1.02); }
  60% { transform: rotate(-3deg) scale(1.01); }
  80% { transform: rotate(3deg) scale(1.01); }
}

.drawn-card {
  border: 1px solid var(--card-color, rgba(255, 255, 255, 0.1));
  border-radius: 24px;
  padding: 44px 40px;
  text-align: center;
  max-width: 300px;
  width: 100%;
  position: relative;
  overflow: hidden;
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2);
}

.card-shine {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: conic-gradient(from 0deg, transparent, rgba(255, 255, 255, 0.03), transparent, rgba(255, 255, 255, 0.03), transparent);
  animation: spin 8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.card-emoji {
  font-size: 56px;
  margin-bottom: 16px;
  position: relative;
}

.card-type {
  font-size: 1.2rem;
  font-weight: 600;
  color: var(--card-color);
  margin-bottom: 12px;
  position: relative;
}

.card-divider {
  width: 40px;
  height: 2px;
  background: var(--card-color);
  opacity: 0.3;
  margin: 0 auto 12px;
  border-radius: 1px;
}

.card-text {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.8);
  line-height: 1.8;
  position: relative;
}

.card-placeholder {
  text-align: center;
  color: rgba(255, 255, 255, 0.3);
}

.placeholder-cards {
  display: flex;
  gap: 8px;
  justify-content: center;
  margin-bottom: 16px;
}

.ph-card {
  width: 50px;
  height: 70px;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  animation: gentleBob 2s ease-in-out infinite;
}

.ph-card:nth-child(2) { animation-delay: -0.5s; }
.ph-card:nth-child(3) { animation-delay: -1s; }

@keyframes gentleBob {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-4px); }
}

.card-placeholder p {
  font-size: 0.8rem;
}

.history {
  margin-top: 40px;
  text-align: center;
}

.history-title {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.3);
  margin-bottom: 8px;
}

.history-items {
  display: flex;
  gap: 8px;
  justify-content: center;
}

.history-item {
  font-size: 20px;
  opacity: 0.5;
}
</style>
