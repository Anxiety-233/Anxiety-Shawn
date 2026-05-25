<script setup>
import { ref, computed } from 'vue'
import gsap from 'gsap'

const cards = [
  { type: '夸夸卡', text: '你今天也超级可爱。', emoji: '🥰', color: '#ff6b9d', rarity: 'N', bg: 'rgba(255, 107, 157, 0.08)' },
  { type: '约会卡', text: '今天一起吃点好吃的。', emoji: '🍜', color: '#ffa726', rarity: 'N', bg: 'rgba(255, 167, 38, 0.08)' },
  { type: '抱抱卡', text: '欠你一个很久很久的抱抱。', emoji: '🤗', color: '#ab47bc', rarity: 'R', bg: 'rgba(171, 71, 188, 0.08)' },
  { type: '想念卡', text: '我现在有点想你。', emoji: '💭', color: '#42a5f5', rarity: 'N', bg: 'rgba(66, 165, 245, 0.08)' },
  { type: '亲亲卡', text: '想亲亲你的额头。', emoji: '😘', color: '#ec407a', rarity: 'R', bg: 'rgba(236, 64, 122, 0.08)' },
  { type: '撒娇卡', text: '今天可以多陪我一会儿吗？', emoji: '🥺', color: '#7e57c2', rarity: 'R', bg: 'rgba(126, 87, 194, 0.08)' },
  { type: '表白卡', text: '我今天也很喜欢你。', emoji: '💗', color: '#e91e63', rarity: 'SR', bg: 'rgba(233, 30, 99, 0.08)' },
  { type: '未来卡', text: '想和你一起变老。', emoji: '🌅', color: '#26a69a', rarity: 'SR', bg: 'rgba(38, 166, 154, 0.08)' },
  { type: '勇气卡', text: '有你在，什么都不怕。', emoji: '💪', color: '#5c6bc0', rarity: 'R', bg: 'rgba(92, 107, 192, 0.08)' },
  { type: '晚安卡', text: '今晚做个有我的梦。', emoji: '🌙', color: '#8d6e63', rarity: 'N', bg: 'rgba(141, 110, 99, 0.08)' },
  { type: '永恒卡', text: '如果有来生，还想遇见你。', emoji: '♾️', color: '#ffd700', rarity: 'SSR', bg: 'rgba(255, 215, 0, 0.06)' },
  { type: '宇宙卡', text: '你是我的整个宇宙。', emoji: '🌌', color: '#e040fb', rarity: 'SSR', bg: 'rgba(224, 64, 251, 0.06)' },
]

const rarityColors = { N: '#aaa', R: '#42a5f5', SR: '#c44dff', SSR: '#ffd700' }
const rarityWeights = { N: 40, R: 30, SR: 20, SSR: 10 }

const currentCard = ref(null)
const isDrawing = ref(false)
const collection = ref(JSON.parse(localStorage.getItem('cardCollection') || '{}'))
const showCollection = ref(false)
const drawCount = ref(parseInt(localStorage.getItem('drawCount') || '0'))

const collectedCount = computed(() => Object.keys(collection.value).length)
const totalCards = computed(() => cards.length)

function weightedDraw() {
  const totalWeight = cards.reduce((s, c) => s + rarityWeights[c.rarity], 0)
  let r = Math.random() * totalWeight
  for (const card of cards) {
    r -= rarityWeights[card.rarity]
    if (r <= 0) return card
  }
  return cards[0]
}

function draw() {
  if (isDrawing.value) return
  isDrawing.value = true
  currentCard.value = null
  drawCount.value++
  localStorage.setItem('drawCount', String(drawCount.value))

  // Pity system: guarantee SSR every 30 draws
  let drawn
  if (drawCount.value % 30 === 0) {
    drawn = cards.filter(c => c.rarity === 'SSR')[Math.floor(Math.random() * 2)]
  } else {
    drawn = weightedDraw()
  }

  setTimeout(() => {
    currentCard.value = drawn
    const isNew = !collection.value[drawn.type]
    collection.value[drawn.type] = (collection.value[drawn.type] || 0) + 1
    localStorage.setItem('cardCollection', JSON.stringify(collection.value))
    isDrawing.value = false

    setTimeout(() => {
      const dur = drawn.rarity === 'SSR' ? 1.2 : drawn.rarity === 'SR' ? 0.9 : 0.7
      gsap.from('.drawn-card', {
        rotateY: 180,
        opacity: 0,
        scale: 0.7,
        duration: dur,
        ease: 'back.out(1.5)'
      })
      if (isNew) {
        gsap.from('.new-badge', { scale: 0, duration: 0.4, delay: 0.5, ease: 'back.out(2)' })
      }
    }, 10)
  }, drawn.rarity === 'SSR' ? 1200 : drawn.rarity === 'SR' ? 1000 : 800)
}
</script>

<template>
  <div class="love-card">
    <h2 class="section-title">恋爱抽卡</h2>
    <p class="card-desc">收集所有甜蜜卡片 · 已收集 {{ collectedCount }}/{{ totalCards }}</p>

    <div class="top-actions">
      <button class="draw-btn" :class="{ drawing: isDrawing }" @click="draw">
        <span class="btn-sparkle">✨</span>
        <span v-if="!isDrawing">抽一张卡</span>
        <span v-else>{{ drawCount % 30 === 0 ? '金光闪闪...' : '抽取中...' }}</span>
      </button>
      <button class="collection-btn" @click="showCollection = !showCollection">
        📖 图鉴
      </button>
    </div>

    <div class="pity-info">
      <span>已抽 {{ drawCount }} 次</span>
      <span>距保底SSR还有 {{ 30 - (drawCount % 30) }} 次</span>
    </div>

    <div v-if="currentCard && !showCollection" class="drawn-card" :style="{ '--card-color': currentCard.color, background: currentCard.bg }">
      <div class="card-shine" :class="currentCard.rarity.toLowerCase()"></div>
      <div class="rarity-badge" :style="{ color: rarityColors[currentCard.rarity] }">{{ currentCard.rarity }}</div>
      <div v-if="collection[currentCard.type] === 1" class="new-badge">NEW!</div>
      <div class="card-emoji">{{ currentCard.emoji }}</div>
      <div class="card-type">{{ currentCard.type }}</div>
      <div class="card-divider"></div>
      <div class="card-text">{{ currentCard.text }}</div>
      <div class="card-count">已拥有 {{ collection[currentCard.type] }} 张</div>
    </div>

    <div v-else-if="!showCollection && !isDrawing" class="card-placeholder">
      <div class="placeholder-cards">
        <div class="ph-card"></div>
        <div class="ph-card"></div>
        <div class="ph-card"></div>
      </div>
      <p>点击按钮抽取卡片</p>
    </div>

    <div v-if="showCollection" class="collection-panel">
      <h3>卡片图鉴</h3>
      <div class="collection-grid">
        <div
          v-for="card in cards"
          :key="card.type"
          class="coll-item"
          :class="{ owned: collection[card.type], [card.rarity.toLowerCase()]: true }"
        >
          <span class="coll-emoji">{{ collection[card.type] ? card.emoji : '❓' }}</span>
          <span class="coll-name">{{ collection[card.type] ? card.type : '???' }}</span>
          <span class="coll-rarity" :style="{ color: rarityColors[card.rarity] }">{{ card.rarity }}</span>
          <span v-if="collection[card.type]" class="coll-count">x{{ collection[card.type] }}</span>
        </div>
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
  margin-bottom: 28px;
}

.top-actions {
  display: flex;
  gap: 12px;
  margin-bottom: 12px;
}

.draw-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 14px 36px;
  border-radius: 30px;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  color: #fff;
  font-size: 1rem;
  font-weight: 500;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 4px 24px rgba(255, 107, 157, 0.3);
  position: relative;
  overflow: hidden;
}

.draw-btn::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, transparent, rgba(255, 255, 255, 0.15), transparent);
  transform: translateX(-100%);
  transition: transform 0.6s;
}

.draw-btn:hover::before { transform: translateX(100%); }
.draw-btn:hover { transform: translateY(-3px); box-shadow: 0 8px 36px rgba(255, 107, 157, 0.45); }
.draw-btn.drawing { animation: shake 0.6s ease-in-out infinite; }

.collection-btn {
  padding: 14px 20px;
  border-radius: 30px;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.9rem;
  transition: all 0.3s;
}

.collection-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.2);
}

.pity-info {
  display: flex;
  gap: 16px;
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.3);
  margin-bottom: 28px;
}

@keyframes shake {
  0%, 100% { transform: rotate(0); }
  25% { transform: rotate(-3deg); }
  75% { transform: rotate(3deg); }
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
  background: conic-gradient(from 0deg, transparent, rgba(255, 255, 255, 0.02), transparent);
  animation: spin 8s linear infinite;
}

.card-shine.ssr {
  background: conic-gradient(from 0deg, transparent, rgba(255, 215, 0, 0.08), transparent, rgba(255, 215, 0, 0.05), transparent);
  animation: spin 3s linear infinite;
}

.card-shine.sr {
  background: conic-gradient(from 0deg, transparent, rgba(196, 77, 255, 0.06), transparent);
  animation: spin 5s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

.rarity-badge {
  position: absolute;
  top: 16px;
  right: 16px;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 1px;
}

.new-badge {
  position: absolute;
  top: 16px;
  left: 16px;
  font-size: 0.7rem;
  font-weight: 600;
  color: #ffd700;
  background: rgba(255, 215, 0, 0.15);
  padding: 2px 8px;
  border-radius: 8px;
}

.card-emoji { font-size: 56px; margin-bottom: 16px; position: relative; }
.card-type { font-size: 1.2rem; font-weight: 600; color: var(--card-color); margin-bottom: 12px; position: relative; }
.card-divider { width: 40px; height: 2px; background: var(--card-color); opacity: 0.3; margin: 0 auto 12px; border-radius: 1px; }
.card-text { font-size: 0.95rem; color: rgba(255, 255, 255, 0.8); line-height: 1.8; position: relative; }
.card-count { font-size: 0.7rem; color: rgba(255, 255, 255, 0.3); margin-top: 12px; position: relative; }

.card-placeholder { text-align: center; color: rgba(255, 255, 255, 0.3); }
.placeholder-cards { display: flex; gap: 8px; justify-content: center; margin-bottom: 16px; }
.ph-card { width: 50px; height: 70px; border-radius: 8px; background: rgba(255, 255, 255, 0.04); border: 1px solid rgba(255, 255, 255, 0.08); animation: gentleBob 2s ease-in-out infinite; }
.ph-card:nth-child(2) { animation-delay: -0.5s; }
.ph-card:nth-child(3) { animation-delay: -1s; }
@keyframes gentleBob { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-4px); } }
.card-placeholder p { font-size: 0.8rem; }

.collection-panel {
  max-width: 450px;
  width: 100%;
}

.collection-panel h3 {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 16px;
  text-align: center;
}

.collection-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(130px, 1fr));
  gap: 10px;
}

.coll-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 14px 10px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.06);
  transition: all 0.3s;
}

.coll-item.owned { border-color: rgba(255, 255, 255, 0.12); }
.coll-item:not(.owned) { opacity: 0.4; }
.coll-item.ssr.owned { border-color: rgba(255, 215, 0, 0.3); background: rgba(255, 215, 0, 0.03); }
.coll-item.sr.owned { border-color: rgba(196, 77, 255, 0.3); background: rgba(196, 77, 255, 0.03); }

.coll-emoji { font-size: 24px; }
.coll-name { font-size: 0.75rem; color: rgba(255, 255, 255, 0.7); }
.coll-rarity { font-size: 0.65rem; font-weight: 600; }
.coll-count { font-size: 0.65rem; color: rgba(255, 255, 255, 0.4); }
</style>
