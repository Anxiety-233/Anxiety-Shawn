<script setup>
import { ref, onMounted, watch } from 'vue'
import gsap from 'gsap'
import HomePage from './components/HomePage.vue'
import LoveCounter from './components/LoveCounter.vue'
import LoveLetter from './components/LoveLetter.vue'
import LoveCard from './components/LoveCard.vue'
import QuizGame from './components/QuizGame.vue'
import CatGame from './components/CatGame.vue'
import CatchGame from './components/CatchGame.vue'
import WishList from './components/WishList.vue'
import Picture from './components/Picture.vue'
import StarBackground from './components/StarBackground.vue'

const currentPage = ref(0)
const entered = ref(false)
const transitioning = ref(false)

const navItems = [
  { label: '计时器', icon: '💫' },
  { label: '情书', icon: '💌' },
  { label: '抽卡', icon: '🎴' },
  { label: '默契', icon: '🎯' },
  { label: '小猫', icon: '🐱' },
  { label: '接爱心', icon: '💝' },
  { label: '愿望', icon: '⭐' },
  { label: '照片', icon: '📷'}
]

function enter() {
  entered.value = true
}

function goTo(page) {
  if (page === currentPage.value || transitioning.value) return
  transitioning.value = true
  gsap.to('.page-content', {
    opacity: 0,
    y: 20,
    duration: 0.25,
    onComplete: () => {
      currentPage.value = page
      gsap.fromTo('.page-content', { opacity: 0, y: -20 }, {
        opacity: 1,
        y: 0,
        duration: 0.35,
        ease: 'power2.out',
        onComplete: () => { transitioning.value = false }
      })
    }
  })
}
</script>

<template>
  <div class="app-container">
    <StarBackground v-if="entered" />
    <HomePage v-if="!entered" @enter="enter" />
    <template v-else>
      <nav class="nav-bar">
        <div class="nav-inner">
          <button
            v-for="(item, i) in navItems"
            :key="i"
            :class="{ active: currentPage === i }"
            @click="goTo(i)"
          >
            <span class="nav-icon">{{ item.icon }}</span>
            <span class="nav-label">{{ item.label }}</span>
          </button>
        </div>
      </nav>
      <div class="page-content">
        <LoveCounter v-if="currentPage === 0" />
        <LoveLetter v-if="currentPage === 1" />
        <LoveCard v-if="currentPage === 2" />
        <QuizGame v-if="currentPage === 3" />
        <CatGame v-if="currentPage === 4" />
        <CatchGame v-if="currentPage === 5" />
        <WishList v-if="currentPage === 6" />
        <Picture v-if="currentPage === 7" />
      </div>
    </template>
  </div>
</template>

<style scoped>
.app-container {
  width: 100%;
  min-height: 100vh;
  position: relative;
  overflow-x: hidden;
}

.nav-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  padding: 14px 16px;
  background: linear-gradient(180deg, rgba(10, 10, 26, 0.95) 0%, rgba(10, 10, 26, 0.8) 100%);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  z-index: 100;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.nav-inner {
  display: flex;
  justify-content: center;
  gap: 4px;
  max-width: 580px;
  margin: 0 auto;
  flex-wrap: wrap;
}

.nav-bar button {
  display: flex;
  align-items: center;
  gap: 3px;
  padding: 7px 11px;
  border-radius: 24px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.06);
  color: rgba(255, 255, 255, 0.55);
  font-size: 12px;
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}

.nav-icon {
  font-size: 14px;
}

.nav-label {
  font-size: 12px;
}

.nav-bar button.active {
  background: linear-gradient(135deg, rgba(255, 107, 157, 0.25), rgba(196, 77, 255, 0.2));
  border-color: rgba(255, 107, 157, 0.35);
  color: #fff;
  box-shadow: 0 2px 16px rgba(255, 107, 157, 0.2), inset 0 1px 0 rgba(255, 255, 255, 0.1);
}

.nav-bar button:hover:not(.active) {
  background: rgba(255, 255, 255, 0.08);
  border-color: rgba(255, 255, 255, 0.12);
  color: rgba(255, 255, 255, 0.85);
  transform: translateY(-1px);
}

.page-content {
  padding-top: 76px;
  min-height: 100vh;
  position: relative;
  z-index: 1;
}
</style>
