<script setup>
import { ref, computed, nextTick } from 'vue'
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

const rarityColors = { N: '#b0bec5', R: '#42a5f5', SR: '#e040fb', SSR: '#ffd700' }
const rarityWeights = { N: 50, R: 30, SR: 15, SSR: 5 }

const currentCards = ref([]) // 改为数组，支持十连
const isDrawing = ref(false)
const collection = ref(JSON.parse(localStorage.getItem('cardCollection') || '{}'))
const showCollection = ref(false)
const drawCount = ref(parseInt(localStorage.getItem('drawCount') || '0'))

const collectedCount = computed(() => Object.keys(collection.value).length)
const totalCards = computed(() => cards.length)
const progressRate = computed(() => (collectedCount.value / totalCards.value) * 100)

function weightedDraw(excludeN = false) {
  let availableCards = cards
  if (excludeN) availableCards = cards.filter(c => c.rarity !== 'N')

  const totalWeight = availableCards.reduce((s, c) => s + rarityWeights[c.rarity], 0)
  let r = Math.random() * totalWeight
  for (const card of availableCards) {
    r -= rarityWeights[card.rarity]
    if (r <= 0) return card
  }
  return availableCards[0]
}

function draw(times = 1) {
  if (isDrawing.value) return
  isDrawing.value = true
  showCollection.value = false
  currentCards.value = []

  // 模拟抽卡加载特效
  setTimeout(() => {
    let results = []
    for (let i = 0; i < times; i++) {
      drawCount.value++
      let drawn

      // 保底机制：30抽必出 SSR
      if (drawCount.value % 30 === 0) {
        drawn = cards.filter(c => c.rarity === 'SSR')[Math.floor(Math.random() * 2)]
      }
      // 十连保底机制：第10张必出 SR 或以上
      else if (times === 10 && i === 9) {
        drawn = weightedDraw(true) // 排除 N 卡
      } else {
        drawn = weightedDraw()
      }

      results.push({ ...drawn, id: Date.now() + i, isNew: !collection.value[drawn.type] })
      collection.value[drawn.type] = (collection.value[drawn.type] || 0) + 1
    }

    localStorage.setItem('drawCount', String(drawCount.value))
    localStorage.setItem('cardCollection', JSON.stringify(collection.value))
    currentCards.value = results

    // GSAP 依次翻牌发牌动画
    nextTick(() => {
      gsap.fromTo('.card-3d-wrapper',
          { rotateY: -180, opacity: 0, scale: 0.5, y: 50 },
          { rotateY: 0, opacity: 1, scale: 1, y: 0, duration: 0.8, stagger: 0.1, ease: 'back.out(1.2)',
            onComplete: () => { isDrawing.value = false }
          }
      )
      gsap.fromTo('.new-badge',
          { scale: 0, rotation: -45 },
          { scale: 1, rotation: -15, duration: 0.5, delay: 0.6, stagger: 0.1, ease: 'back.out(2)' }
      )
    })
  }, 800)
}

// 3D 卡片跟随鼠标倾斜效果
function handleMouseMove(e, index) {
  const card = document.getElementById(`card-${index}`)
  if (!card) return
  const rect = card.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  const centerX = rect.width / 2
  const centerY = rect.height / 2
  const rotateX = ((y - centerY) / centerY) * -15 // 最大倾斜15度
  const rotateY = ((x - centerX) / centerX) * 15

  card.style.setProperty('--rx', `${rotateX}deg`)
  card.style.setProperty('--ry', `${rotateY}deg`)
  card.style.setProperty('--tx', `${(x / rect.width) * 100}%`)
  card.style.setProperty('--ty', `${(y / rect.height) * 100}%`)
}

function handleMouseLeave(index) {
  const card = document.getElementById(`card-${index}`)
  if (!card) return
  card.style.setProperty('--rx', '0deg')
  card.style.setProperty('--ry', '0deg')
  card.style.setProperty('--tx', '50%')
  card.style.setProperty('--ty', '50%')
}
</script>

<template>
  <div class="love-card">
    <div class="header-section">
      <h2 class="section-title">恋爱集卡册</h2>

      <div class="progress-container">
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: `${progressRate}%` }"></div>
        </div>
        <div class="progress-text">收集度 {{ collectedCount }} / {{ totalCards }}</div>
      </div>
    </div>

    <div class="top-actions">
      <button class="draw-btn single-btn" :class="{ disabled: isDrawing }" @click="draw(1)">
        <span class="btn-icon">💌</span> 单抽
      </button>
      <button class="draw-btn ten-btn" :class="{ disabled: isDrawing }" @click="draw(10)">
        <span class="btn-icon">✨</span> 十连抽
        <div class="btn-tag">必出SR</div>
      </button>
      <button class="collection-btn" @click="showCollection = !showCollection">
        📖 {{ showCollection ? '返回抽卡' : '图鉴' }}
      </button>
    </div>

    <div class="pity-info">
      <div class="pity-track">
        <span>💫 总计抽取: <strong>{{ drawCount }}</strong> 次</span>
        <span class="pity-highlight">距必出SSR还有 <strong>{{ 30 - (drawCount % 30) }}</strong> 次</span>
      </div>
    </div>

    <div v-if="currentCards.length > 0 && !showCollection" class="results-area" :class="{'multi-pull': currentCards.length > 1}">
      <div
          v-for="(card, index) in currentCards"
          :key="card.id"
          class="card-3d-wrapper"
          @mousemove="handleMouseMove($event, index)"
          @mouseleave="handleMouseLeave(index)"
      >
        <div :id="`card-${index}`" class="drawn-card" :class="[card.rarity.toLowerCase()]" :style="{ '--card-color': card.color, background: card.bg }">

          <div v-if="['SR', 'SSR'].includes(card.rarity)" class="holo-glare"></div>

          <div class="card-shine" :class="card.rarity.toLowerCase()"></div>

          <div class="rarity-badge" :style="{ color: rarityColors[card.rarity] }">{{ card.rarity }}</div>
          <div v-if="card.isNew" class="new-badge">NEW</div>

          <div class="card-emoji">{{ card.emoji }}</div>
          <div class="card-type">{{ card.type }}</div>
          <div class="card-divider"></div>

          <div v-if="currentCards.length === 1" class="card-text">{{ card.text }}</div>
          <div class="card-count">已拥有 {{ collection[card.type] }} 张</div>
        </div>
      </div>
    </div>

    <div v-else-if="!showCollection && !isDrawing" class="card-placeholder">
      <div class="magic-circle">🔯</div>
      <p>呼唤爱的奇迹...</p>
    </div>

    <div v-if="isDrawing" class="drawing-animation">
      <div class="sparkles">✨🌟✨</div>
      <p>{{ currentCards.length === 10 ? '奇迹汇聚中 (十连)...' : '感应中...' }}</p>
    </div>

    <div v-if="showCollection" class="collection-panel">
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
  padding: 30px 15px;
  background: radial-gradient(circle at top, #1a1525, #0a0a10);
  font-family: 'PingFang SC', sans-serif;
  overflow-x: hidden;
}

/* 头部进度条 */
.header-section { text-align: center; margin-bottom: 25px; width: 100%; max-width: 400px; }
.section-title {
  font-size: 1.8rem; font-weight: 800; margin-bottom: 12px;
  background: linear-gradient(135deg, #fff, #ff9a9e);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
}
.progress-container { background: rgba(255,255,255,0.05); padding: 12px 20px; border-radius: 16px; border: 1px solid rgba(255,255,255,0.08); }
.progress-bar { width: 100%; height: 8px; background: rgba(0,0,0,0.4); border-radius: 4px; overflow: hidden; margin-bottom: 8px; }
.progress-fill { height: 100%; background: linear-gradient(90deg, #ff6b9d, #ffd700); transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1); }
.progress-text { font-size: 0.8rem; color: #a0a0b0; font-weight: 600; }

/* 按钮区 */
.top-actions { display: flex; gap: 10px; margin-bottom: 15px; flex-wrap: wrap; justify-content: center; }
.draw-btn {
  position: relative; display: inline-flex; align-items: center; gap: 6px; padding: 12px 24px;
  border-radius: 25px; border: none; color: #fff; font-weight: 600; cursor: pointer; transition: all 0.3s;
}
.single-btn { background: rgba(255, 255, 255, 0.1); border: 1px solid rgba(255, 255, 255, 0.2); }
.single-btn:hover:not(.disabled) { background: rgba(255, 255, 255, 0.2); transform: translateY(-2px); }
.ten-btn { background: linear-gradient(135deg, #ff6b9d, #c44dff); box-shadow: 0 4px 20px rgba(196, 77, 255, 0.4); padding-right: 32px; }
.ten-btn:hover:not(.disabled) { transform: translateY(-2px); box-shadow: 0 6px 25px rgba(196, 77, 255, 0.6); }
.btn-tag { position: absolute; top: -8px; right: -5px; background: #ffd700; color: #000; font-size: 0.65rem; padding: 2px 6px; border-radius: 8px; font-weight: 800; transform: rotate(10deg); box-shadow: 0 2px 5px rgba(0,0,0,0.3); }
.collection-btn { padding: 12px 20px; border-radius: 25px; background: transparent; color: #a0a0b0; border: 1px dashed rgba(255,255,255,0.3); cursor: pointer; transition: all 0.3s; }
.collection-btn:hover { color: #fff; border-color: #fff; }
.disabled { opacity: 0.6; pointer-events: none; }

.pity-info { margin-bottom: 30px; }
.pity-track { display: flex; gap: 15px; font-size: 0.8rem; color: #888; background: rgba(0,0,0,0.3); padding: 8px 20px; border-radius: 20px; }
.pity-track strong { color: #fff; }
.pity-highlight strong { color: #ffd700; }

/* 3D 卡片基础包装 */
.results-area { perspective: 1000px; display: flex; justify-content: center; width: 100%; }
.card-3d-wrapper { transform-style: preserve-3d; }

/* 单抽大卡片样式 */
.drawn-card {
  --rx: 0deg; --ry: 0deg; --tx: 50%; --ty: 50%;
  transform: rotateX(var(--rx)) rotateY(var(--ry));
  transition: transform 0.1s ease-out; /* 鼠标移动时的平滑度 */
  border: 1px solid var(--card-color, rgba(255, 255, 255, 0.1));
  border-radius: 20px; padding: 40px 30px; text-align: center;
  width: 280px; position: relative; overflow: hidden;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.4), inset 0 0 0 1px rgba(255,255,255,0.1);
  background-color: #1a1a24; /* 兜底底色 */
}

/* 十连抽网格布局与微缩卡片 */
.results-area.multi-pull {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(130px, 1fr)); gap: 15px;
  max-width: 800px; width: 100%; align-items: center; justify-items: center;
}
.multi-pull .drawn-card { width: 140px; padding: 25px 10px; border-radius: 16px; }
.multi-pull .card-emoji { font-size: 36px; margin-bottom: 8px; }
.multi-pull .card-type { font-size: 1rem; margin-bottom: 5px; }

/* 全息镭射光泽特效 (仅高稀有度) */
.holo-glare {
  position: absolute; inset: 0; pointer-events: none; mix-blend-mode: color-dodge; opacity: 0.6;
  background: radial-gradient(circle at var(--tx) var(--ty),
  rgba(255,255,255,0.8) 0%, rgba(255,192,203,0.4) 20%,
  rgba(135,206,235,0.2) 40%, transparent 60%);
  z-index: 2; transition: background 0.1s;
}
.drawn-card.ssr .holo-glare {
  background: radial-gradient(circle at var(--tx) var(--ty),
  rgba(255,255,255,1) 0%, rgba(255,215,0,0.6) 15%,
  rgba(255,105,180,0.4) 30%, rgba(0,255,255,0.2) 45%, transparent 65%);
}

/* 卡片内部元素 */
.rarity-badge { position: absolute; top: 12px; right: 12px; font-size: 0.8rem; font-weight: 800; font-style: italic; text-shadow: 0 2px 4px rgba(0,0,0,0.5); z-index: 3; }
.new-badge { position: absolute; top: -10px; left: -25px; font-size: 0.7rem; font-weight: 800; color: #fff; background: #ff3366; padding: 15px 25px 5px; transform: rotate(-45deg); z-index: 3; box-shadow: 0 2px 10px rgba(255,51,102,0.5); }
.card-emoji { font-size: 64px; margin-bottom: 20px; position: relative; z-index: 3; filter: drop-shadow(0 4px 10px rgba(0,0,0,0.3)); }
.card-type { font-size: 1.3rem; font-weight: 800; color: var(--card-color); position: relative; z-index: 3; }
.card-divider { width: 30px; height: 3px; background: var(--card-color); margin: 12px auto; border-radius: 2px; z-index: 3; position: relative; }
.card-text { font-size: 0.95rem; color: #e0e0e0; line-height: 1.6; z-index: 3; position: relative; font-weight: 500; }
.card-count { font-size: 0.75rem; color: rgba(255, 255, 255, 0.4); margin-top: 15px; z-index: 3; position: relative; }

/* 占位符与动画提示 */
.card-placeholder { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 300px; opacity: 0.5; }
.magic-circle { font-size: 80px; animation: slowSpin 10s linear infinite; margin-bottom: 20px; color: #ff6b9d; }
.drawing-animation { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 300px; color: #fff; }
.sparkles { font-size: 40px; animation: bounce 0.6s infinite alternate; margin-bottom: 15px; }

@keyframes slowSpin { to { transform: rotate(360deg); } }
@keyframes bounce { from { transform: translateY(0); } to { transform: translateY(-15px); } }

/* 图鉴系统样式 (优化网格) */
.collection-panel { width: 100%; max-width: 600px; background: rgba(0,0,0,0.2); padding: 20px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.05); }
.collection-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(100px, 1fr)); gap: 12px; }
.coll-item {
  display: flex; flex-direction: column; align-items: center; gap: 6px; padding: 15px 10px;
  border-radius: 12px; background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.06); transition: all 0.3s;
}
.coll-item.owned { border-color: rgba(255, 255, 255, 0.15); background: rgba(255, 255, 255, 0.08); box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
.coll-item:not(.owned) { opacity: 0.3; filter: grayscale(100%); }
.coll-item.ssr.owned { border-color: rgba(255, 215, 0, 0.4); background: rgba(255, 215, 0, 0.05); }
.coll-item.sr.owned { border-color: rgba(196, 77, 255, 0.4); background: rgba(196, 77, 255, 0.05); }
.coll-emoji { font-size: 28px; filter: drop-shadow(0 2px 5px rgba(0,0,0,0.2)); }
.coll-name { font-size: 0.75rem; color: #e0e0e0; font-weight: 600; }
.coll-rarity { font-size: 0.7rem; font-weight: 800; font-style: italic; }
.coll-count { font-size: 0.7rem; color: #aaa; background: rgba(0,0,0,0.3); padding: 2px 8px; border-radius: 10px; }
</style>