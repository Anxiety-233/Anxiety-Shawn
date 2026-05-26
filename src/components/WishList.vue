<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// 默认愿望库
const defaultWishes = [
  { id: '1', text: '一起看一次日出', icon: '🌅', done: false },
  { id: '2', text: '一起吃一顿很撑的火锅', icon: '🍲', done: false },
  { id: '3', text: '一起去一个没去过的地方', icon: '✈️', done: false },
  { id: '4', text: '一起拍好多好多照片', icon: '📸', done: false },
  { id: '5', text: '一起过很多个生日', icon: '🎂', done: false },
  { id: '6', text: '一起养一只小猫', icon: '🐱', done: false },
  { id: '7', text: '一起在雨天窝在家里', icon: '🌧️', done: false },
  { id: '8', text: '一起看一场烟花', icon: '🎆', done: false },
  { id: '9', text: '一起走过四季', icon: '🍂', done: false },
  { id: '10', text: '一起白头到老', icon: '👫', done: false },
]

// 状态管理
const wishes = ref([])
const newWishText = ref('')
const hearts = ref([])
let heartId = 0

// 抽签与特效状态
const isPicking = ref(false)
const highlightedId = ref(null)

// 初始化与持久化
onMounted(() => {
  const saved = localStorage.getItem('couple_wishes')
  if (saved) {
    wishes.value = JSON.parse(saved)
  } else {
    wishes.value = [...defaultWishes]
  }
})

watch(wishes, (newVal) => {
  localStorage.setItem('couple_wishes', JSON.stringify(newVal))
}, { deep: true })

// 计算属性
const completedCount = computed(() => wishes.value.filter(w => w.done).length)
const progress = computed(() => wishes.value.length === 0 ? 0 : Math.round((completedCount.value / wishes.value.length) * 100))
const allDone = computed(() => wishes.value.length > 0 && completedCount.value === wishes.value.length)

const milestoneTitle = computed(() => {
  const p = progress.value
  if (p === 0) return '旅程的开始 启航'
  if (p < 30) return '慢慢靠近 怦然心动'
  if (p < 60) return '渐入佳境 灵魂伴侣'
  if (p < 100) return '默契满分 命中注定'
  return '完美恋人 往后余生'
})

// 交互操作
function toggleWish(wish) {
  if (isPicking.value) return
  wish.done = !wish.done

  if (wish.done) {
    // 撒花特效
    for (let i = 0; i < 5; i++) {
      const id = heartId++
      hearts.value.push({
        id,
        x: Math.random() * 80 + 10,
        size: Math.random() * 15 + 10,
        delay: i * 0.1,
        tx: (Math.random() - 0.5) * 100 // 随机水平飘动
      })
      setTimeout(() => { hearts.value = hearts.value.filter(h => h.id !== id) }, 2500)
    }
  }
}

function addWish() {
  const text = newWishText.value.trim()
  if (!text) return
  const icons = ['✨', '💫', '🌟', '💝', '🎈', '🎡', '🎨', '🎵']
  const randomIcon = icons[Math.floor(Math.random() * icons.length)]

  wishes.value.unshift({
    id: Date.now().toString(),
    text,
    icon: randomIcon,
    done: false
  })
  newWishText.value = ''
}

function removeWish(id) {
  if (isPicking.value) return
  wishes.value = wishes.value.filter(w => w.id !== id)
}

// 随机抽签系统
function pickRandomWish() {
  const pending = wishes.value.filter(w => !w.done)
  if (pending.length === 0 || isPicking.value) return

  isPicking.value = true
  let jumps = 0
  const maxJumps = 20
  let speed = 50

  const jump = () => {
    const randomWish = pending[Math.floor(Math.random() * pending.length)]
    highlightedId.value = randomWish.id
    jumps++

    if (jumps < maxJumps) {
      speed += 10 // 逐渐减速
      setTimeout(jump, speed)
    } else {
      isPicking.value = false
      // 最终选定特效
      setTimeout(() => { highlightedId.value = null }, 3000)
    }
  }
  jump()
}
</script>

<template>
  <div class="wish-list">
    <div class="header-section">
      <h2 class="section-title">我们的愿望清单</h2>
      <p class="section-sub">点亮每一个想和你一起完成的瞬间</p>

      <div class="milestone-badge">{{ milestoneTitle }}</div>

      <div class="progress-section">
        <div class="progress-bar-bg">
          <div class="progress-bar-fill" :style="{ width: progress + '%' }"></div>
        </div>
        <span class="progress-text">{{ completedCount }} / {{ wishes.length }}</span>
      </div>
    </div>

    <div class="control-panel">
      <div class="input-group">
        <input
            v-model="newWishText"
            @keyup.enter="addWish"
            type="text"
            placeholder="写下新的愿望..."
            maxlength="30"
        />
        <button class="add-btn" @click="addWish" :disabled="!newWishText.trim()">添加</button>
      </div>
      <button class="random-btn" @click="pickRandomWish" :disabled="isPicking || completedCount === wishes.length">
        🎰 随机抽一个
      </button>
    </div>

    <div class="wishes-container">
      <transition-group name="list" tag="div" class="wishes">
        <div
            v-for="(wish, i) in wishes"
            :key="wish.id"
            class="wish-item"
            :class="{
            done: wish.done,
            highlighted: highlightedId === wish.id
          }"
            :style="{ animationDelay: (i * 0.05) + 's' }"
            @click="toggleWish(wish)"
        >
          <span class="wish-icon">{{ wish.icon }}</span>

          <div class="wish-content">
            <span class="wish-text">{{ wish.text }}</span>
            <div class="strike-line"></div>
          </div>

          <span class="wish-status">
            <transition name="pop" mode="out-in">
              <span v-if="wish.done" class="wish-heart">❤️</span>
              <span v-else class="wish-circle"></span>
            </transition>
          </span>

          <button class="delete-btn" @click.stop="removeWish(wish.id)">×</button>
        </div>
      </transition-group>
    </div>

    <div class="floating-hearts">
      <div
          v-for="heart in hearts"
          :key="heart.id"
          class="float-heart"
          :style="{
          left: heart.x + '%',
          fontSize: heart.size + 'px',
          animationDelay: heart.delay + 's',
          '--tx': heart.tx + 'px'
        }"
      >❤️</div>
    </div>

    <transition name="fade">
      <div v-if="allDone" class="completion-card glass-panel">
        <div class="completion-emoji">🌟</div>
        <h3 class="completion-title">清单已完成</h3>
        <p class="completion-text">这只是我们故事的序章<br>继续添加新的愿望吧！</p>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.wish-list {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 76px);
  padding: 40px 20px 80px;
  background: radial-gradient(circle at top, #1a1525, #0a0a10);
  font-family: 'PingFang SC', sans-serif;
  overflow-x: hidden;
}

/* 头部样式 */
.header-section { text-align: center; width: 100%; max-width: 450px; margin-bottom: 25px; }
.section-title { font-size: 1.8rem; font-weight: 800; margin-bottom: 8px; background: linear-gradient(135deg, #fff, #ff9a9e); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.section-sub { font-size: 0.85rem; color: rgba(255, 255, 255, 0.5); margin-bottom: 15px; }
.milestone-badge { display: inline-block; padding: 4px 12px; background: rgba(196, 77, 255, 0.15); border: 1px solid rgba(196, 77, 255, 0.3); border-radius: 20px; color: #e0b0ff; font-size: 0.75rem; font-weight: 600; margin-bottom: 20px; letter-spacing: 1px; }

/* 进度条 */
.progress-section { display: flex; align-items: center; gap: 15px; width: 100%; }
.progress-bar-bg { flex: 1; height: 6px; background: rgba(255, 255, 255, 0.05); border-radius: 3px; overflow: hidden; box-shadow: inset 0 1px 3px rgba(0,0,0,0.3); }
.progress-bar-fill { height: 100%; background: linear-gradient(90deg, #ff6b9d, #c44dff); border-radius: 3px; transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1); box-shadow: 0 0 10px rgba(196, 77, 255, 0.5); }
.progress-text { font-size: 0.85rem; font-weight: 600; color: #fff; min-width: 45px; text-align: right; }

/* 控制面板 */
.control-panel { width: 100%; max-width: 450px; display: flex; flex-direction: column; gap: 12px; margin-bottom: 30px; }
.input-group { display: flex; gap: 8px; }
.input-group input { flex: 1; background: rgba(255, 255, 255, 0.05); border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 12px; padding: 12px 16px; color: #fff; font-size: 0.9rem; outline: none; transition: all 0.3s; }
.input-group input:focus { border-color: rgba(255, 107, 157, 0.5); background: rgba(255, 255, 255, 0.08); }
.add-btn { background: rgba(255, 107, 157, 0.15); border: 1px solid rgba(255, 107, 157, 0.3); color: #ff9a9e; border-radius: 12px; padding: 0 20px; font-weight: 600; cursor: pointer; transition: all 0.3s; }
.add-btn:not(:disabled):hover { background: rgba(255, 107, 157, 0.3); }
.add-btn:disabled { opacity: 0.5; cursor: not-allowed; }
.random-btn { background: linear-gradient(135deg, rgba(196, 77, 255, 0.2), rgba(255, 107, 157, 0.2)); border: 1px dashed rgba(255, 107, 157, 0.4); color: #fff; border-radius: 12px; padding: 12px; font-size: 0.9rem; font-weight: 600; cursor: pointer; transition: all 0.3s; }
.random-btn:not(:disabled):hover { background: rgba(196, 77, 255, 0.4); transform: translateY(-2px); box-shadow: 0 4px 15px rgba(196, 77, 255, 0.2); }

/* 列表样式 */
.wishes-container { width: 100%; max-width: 450px; }
.wishes { display: flex; flex-direction: column; gap: 12px; }

.wish-item {
  position: relative; display: flex; align-items: center; gap: 15px; padding: 18px 20px;
  border-radius: 16px; background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.05);
  cursor: pointer; transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1); overflow: hidden;
}
.wish-item:hover { background: rgba(255, 255, 255, 0.06); transform: translateY(-2px); }

/* 随机抽取高亮特效 */
.wish-item.highlighted {
  background: rgba(255, 215, 0, 0.15); border-color: #ffd700;
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.3); transform: scale(1.02); z-index: 10;
}

/* 完成状态样式 */
.wish-item.done { background: rgba(255, 107, 157, 0.05); border-color: rgba(255, 107, 157, 0.2); opacity: 0.85; }
.wish-icon { font-size: 1.4rem; width: 32px; text-align: center; z-index: 2; }
.wish-content { flex: 1; position: relative; z-index: 2; }
.wish-text { font-size: 0.95rem; color: #e0e0e0; transition: color 0.3s; }
.wish-item.done .wish-text { color: rgba(255, 255, 255, 0.4); }

/* 发光划线特效 */
.strike-line {
  position: absolute; top: 50%; left: 0; height: 2px; width: 0;
  background: linear-gradient(90deg, #ff6b9d, #c44dff, transparent);
  border-radius: 2px; box-shadow: 0 0 8px rgba(255, 107, 157, 0.8);
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1); pointer-events: none;
}
.wish-item.done .strike-line { width: 100%; }

/* 状态图标 */
.wish-status { width: 28px; display: flex; align-items: center; justify-content: center; z-index: 2; }
.wish-circle { width: 20px; height: 20px; border-radius: 50%; border: 2px solid rgba(255, 255, 255, 0.15); transition: all 0.3s; }
.wish-item:hover .wish-circle { border-color: rgba(255, 107, 157, 0.5); }
.wish-heart { font-size: 18px; filter: drop-shadow(0 0 5px rgba(255, 107, 157, 0.5)); }

/* 删除按钮 */
.delete-btn {
  position: absolute; right: 10px; top: 50%; transform: translateY(-50%) scale(0.8);
  width: 30px; height: 30px; border-radius: 50%; background: rgba(255, 50, 50, 0.2);
  color: #ff8c8c; border: none; font-size: 1.2rem; cursor: pointer;
  opacity: 0; transition: all 0.2s; z-index: 3;
}
.wish-item:hover .delete-btn { opacity: 1; transform: translateY(-50%) scale(1); right: -10px; } /* 悬浮时滑出 */
.wish-item:hover .wish-status { transform: translateX(-30px); } /* 为删除键让路 */

/* 列表过渡动画 */
.list-enter-active, .list-leave-active { transition: all 0.5s ease; }
.list-enter-from { opacity: 0; transform: translateY(-20px) scale(0.9); }
.list-leave-to { opacity: 0; transform: translateX(50px); }

/* 心跳动画 */
.pop-enter-active { animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
@keyframes popIn { 0% { transform: scale(0) rotate(-30deg); } 50% { transform: scale(1.3) rotate(10deg); } 100% { transform: scale(1) rotate(0); } }

/* 漂浮爱心 */
.floating-hearts { position: fixed; inset: 0; pointer-events: none; z-index: 50; }
.float-heart { position: absolute; bottom: 20%; animation: floatUp 2.5s ease-out forwards; filter: drop-shadow(0 0 8px rgba(255, 107, 157, 0.6)); }
@keyframes floatUp {
  0% { transform: translate(0, 0) scale(0.5); opacity: 0; }
  20% { opacity: 1; transform: translate(calc(var(--tx) * 0.2), -20vh) scale(1.2); }
  100% { transform: translate(var(--tx), -70vh) scale(0.8) rotate(20deg); opacity: 0; }
}

/* 庆祝卡片 */
.completion-card {
  position: fixed; bottom: 40px; left: 50%; transform: translateX(-50%);
  text-align: center; padding: 25px 40px; border-radius: 20px; width: 90%; max-width: 400px;
  background: rgba(15, 10, 30, 0.85); backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 215, 0, 0.3); box-shadow: 0 15px 40px rgba(0,0,0,0.5), 0 0 30px rgba(255, 215, 0, 0.1);
  z-index: 100;
}
.completion-emoji { font-size: 40px; margin-bottom: 5px; animation: gentleBob 2s infinite ease-in-out; }
.completion-title { color: #ffd700; font-size: 1.2rem; margin-bottom: 8px; }
.completion-text { font-size: 0.9rem; color: rgba(255, 255, 255, 0.7); line-height: 1.6; }
@keyframes gentleBob { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-8px); } }

.fade-enter-active, .fade-leave-active { transition: all 0.6s cubic-bezier(0.4, 0, 0.2, 1); }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translate(-50%, 40px) scale(0.9); }
</style>