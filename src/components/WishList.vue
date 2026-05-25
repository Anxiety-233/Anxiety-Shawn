<script setup>
import { ref, computed } from 'vue'

const wishes = [
  { text: '一起看一次日出', icon: '🌅' },
  { text: '一起吃一顿很撑的火锅', icon: '🍲' },
  { text: '一起去一个没去过的地方', icon: '✈️' },
  { text: '一起拍好多好多照片', icon: '📸' },
  { text: '一起过很多个生日', icon: '🎂' },
  { text: '一起养一只小猫', icon: '🐱' },
  { text: '一起在雨天窝在家里', icon: '🌧️' },
  { text: '一起看一场烟花', icon: '🎆' },
  { text: '一起走过四季', icon: '🍂' },
  { text: '一起白头到老', icon: '👫' },
]

const clickedWishes = ref(new Set())
const hearts = ref([])
let heartId = 0

const progress = computed(() => Math.round(clickedWishes.value.size / wishes.length * 100))
const allDone = computed(() => clickedWishes.value.size === wishes.length)

function toggleWish(idx) {
  if (clickedWishes.value.has(idx)) return
  clickedWishes.value = new Set([...clickedWishes.value, idx])

  for (let i = 0; i < 3; i++) {
    const id = heartId++
    hearts.value.push({ id, x: Math.random() * 80 + 10, size: Math.random() * 12 + 10, delay: i * 0.15 })
    setTimeout(() => {
      hearts.value = hearts.value.filter(h => h.id !== id)
    }, 2500)
  }
}
</script>

<template>
  <div class="wish-list">
    <h2 class="section-title">我们的愿望清单</h2>
    <p class="section-sub">点亮每一个想和你一起完成的愿望</p>

    <div class="progress-section">
      <div class="progress-bar-bg">
        <div class="progress-bar-fill" :style="{ width: progress + '%' }"></div>
      </div>
      <span class="progress-text">{{ clickedWishes.size }} / {{ wishes.length }}</span>
    </div>

    <div class="wishes">
      <div
        v-for="(wish, i) in wishes"
        :key="i"
        class="wish-item"
        :class="{ done: clickedWishes.has(i) }"
        :style="{ animationDelay: (i * 0.05) + 's' }"
        @click="toggleWish(i)"
      >
        <span class="wish-icon">{{ wish.icon }}</span>
        <span class="wish-text">{{ wish.text }}</span>
        <span class="wish-status">
          <span v-if="clickedWishes.has(i)" class="wish-heart">❤️</span>
          <span v-else class="wish-circle"></span>
        </span>
      </div>
    </div>

    <div class="floating-hearts">
      <div
        v-for="heart in hearts"
        :key="heart.id"
        class="float-heart"
        :style="{ left: heart.x + '%', fontSize: heart.size + 'px', animationDelay: heart.delay + 's' }"
      >❤️</div>
    </div>

    <transition name="fade">
      <div v-if="allDone" class="completion-card">
        <p class="completion-emoji">🌟</p>
        <p class="completion-text">所有愿望都会实现的<br>因为有你在身边</p>
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
  padding: 40px 20px;
  position: relative;
  overflow: hidden;
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
  margin-bottom: 24px;
}

.progress-section {
  display: flex;
  align-items: center;
  gap: 12px;
  width: 100%;
  max-width: 400px;
  margin-bottom: 28px;
}

.progress-bar-bg {
  flex: 1;
  height: 4px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 2px;
  overflow: hidden;
}

.progress-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #ff6b9d, #c44dff);
  border-radius: 2px;
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 0 6px rgba(255, 107, 157, 0.4);
}

.progress-text {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.4);
  min-width: 40px;
  text-align: right;
}

.wishes {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-width: 420px;
  width: 100%;
}

.wish-item {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 16px 20px;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.06);
  cursor: pointer;
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  animation: slideIn 0.4s ease both;
}

@keyframes slideIn {
  from { opacity: 0; transform: translateX(-10px); }
  to { opacity: 1; transform: translateX(0); }
}

.wish-item:hover:not(.done) {
  background: rgba(255, 107, 157, 0.06);
  border-color: rgba(255, 107, 157, 0.15);
  transform: translateX(4px);
}

.wish-item.done {
  background: linear-gradient(135deg, rgba(255, 107, 157, 0.08), rgba(196, 77, 255, 0.05));
  border-color: rgba(255, 107, 157, 0.2);
}

.wish-icon {
  font-size: 1.3rem;
  width: 32px;
  text-align: center;
}

.wish-text {
  flex: 1;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.75);
}

.wish-status {
  width: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.wish-circle {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 1.5px solid rgba(255, 255, 255, 0.15);
  transition: all 0.3s;
}

.wish-item:hover .wish-circle {
  border-color: rgba(255, 107, 157, 0.4);
}

.wish-heart {
  animation: pop 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  font-size: 16px;
}

@keyframes pop {
  0% { transform: scale(0) rotate(-30deg); }
  60% { transform: scale(1.4) rotate(5deg); }
  100% { transform: scale(1) rotate(0); }
}

.floating-hearts {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 50;
}

.float-heart {
  position: absolute;
  bottom: 20%;
  animation: floatUp 2.5s ease-out forwards;
}

@keyframes floatUp {
  0% {
    transform: translateY(0) scale(1) rotate(0deg);
    opacity: 1;
  }
  100% {
    transform: translateY(-60vh) scale(0.3) rotate(20deg);
    opacity: 0;
  }
}

.completion-card {
  margin-top: 32px;
  text-align: center;
  padding: 24px;
  border-radius: 16px;
  background: rgba(255, 107, 157, 0.06);
  border: 1px solid rgba(255, 107, 157, 0.15);
}

.completion-emoji {
  font-size: 32px;
  margin-bottom: 12px;
}

.completion-text {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.6);
  line-height: 1.8;
}

.fade-enter-active { transition: all 0.6s ease; }
.fade-enter-from { opacity: 0; transform: translateY(20px); }
</style>
