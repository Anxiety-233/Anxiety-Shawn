<script setup>
import { ref } from 'vue'
import gsap from 'gsap'

const wishes = [
  { text: '一起看一次日出', icon: '🌅' },
  { text: '一起吃一顿很撑的火锅', icon: '🍲' },
  { text: '一起去一个没去过的地方', icon: '✈️' },
  { text: '一起拍好多好多照片', icon: '📸' },
  { text: '一起过很多个生日', icon: '🎂' },
  { text: '一起养一只小猫', icon: '🐱' },
  { text: '一起在雨天窝在家里', icon: '🌧️' },
  { text: '一起白头到老', icon: '👫' },
]

const clickedWishes = ref(new Set())
const hearts = ref([])
let heartId = 0

function toggleWish(idx) {
  if (clickedWishes.value.has(idx)) return
  clickedWishes.value.add(idx)

  const id = heartId++
  hearts.value.push({ id, x: Math.random() * 80 + 10, delay: 0 })

  setTimeout(() => {
    hearts.value = hearts.value.filter(h => h.id !== id)
  }, 2000)
}
</script>

<template>
  <div class="wish-list">
    <h2 class="section-title">我们的愿望清单</h2>
    <p class="wish-desc">点亮每一个想和你一起完成的愿望</p>

    <div class="wishes">
      <div
        v-for="(wish, i) in wishes"
        :key="i"
        class="wish-item"
        :class="{ done: clickedWishes.has(i) }"
        @click="toggleWish(i)"
      >
        <span class="wish-icon">{{ wish.icon }}</span>
        <span class="wish-text">{{ wish.text }}</span>
        <span class="wish-heart" v-if="clickedWishes.has(i)">❤️</span>
      </div>
    </div>

    <div class="floating-hearts">
      <div
        v-for="heart in hearts"
        :key="heart.id"
        class="float-heart"
        :style="{ left: heart.x + '%' }"
      >❤️</div>
    </div>

    <p class="wish-footer" v-if="clickedWishes.size === wishes.length">
      所有愿望都会实现的，因为有你在身边 💫
    </p>
  </div>
</template>

<style scoped>
.wish-list {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 70px);
  padding: 40px 20px;
  position: relative;
  overflow: hidden;
}

.section-title {
  font-size: 1.5rem;
  margin-bottom: 12px;
  color: rgba(255, 255, 255, 0.9);
}

.wish-desc {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.5);
  margin-bottom: 32px;
}

.wishes {
  display: flex;
  flex-direction: column;
  gap: 12px;
  max-width: 400px;
  width: 100%;
}

.wish-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px 20px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  cursor: pointer;
  transition: all 0.3s;
}

.wish-item:hover {
  background: rgba(255, 107, 157, 0.08);
  border-color: rgba(255, 107, 157, 0.2);
}

.wish-item.done {
  background: rgba(255, 107, 157, 0.1);
  border-color: rgba(255, 107, 157, 0.3);
}

.wish-icon {
  font-size: 1.3rem;
}

.wish-text {
  flex: 1;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.8);
}

.wish-heart {
  animation: pop 0.3s ease;
}

@keyframes pop {
  0% { transform: scale(0); }
  50% { transform: scale(1.3); }
  100% { transform: scale(1); }
}

.floating-hearts {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 50;
}

.float-heart {
  position: absolute;
  bottom: 0;
  font-size: 20px;
  animation: floatUp 2s ease-out forwards;
}

@keyframes floatUp {
  0% {
    transform: translateY(0) scale(1);
    opacity: 1;
  }
  100% {
    transform: translateY(-100vh) scale(0.5);
    opacity: 0;
  }
}

.wish-footer {
  margin-top: 32px;
  font-size: 0.9rem;
  color: rgba(255, 107, 157, 0.8);
  text-align: center;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>
