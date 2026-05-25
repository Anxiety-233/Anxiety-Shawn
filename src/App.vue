<script setup>
import { ref } from 'vue'
import HomePage from './components/HomePage.vue'
import LoveCounter from './components/LoveCounter.vue'
import LoveLetter from './components/LoveLetter.vue'
import LoveCard from './components/LoveCard.vue'
import QuizGame from './components/QuizGame.vue'
import CatGame from './components/CatGame.vue'
import WishList from './components/WishList.vue'

const currentPage = ref(0)
const entered = ref(false)

function enter() {
  entered.value = true
}

function goTo(page) {
  currentPage.value = page
}
</script>

<template>
  <div class="app-container">
    <HomePage v-if="!entered" @enter="enter" />
    <template v-else>
      <nav class="nav-bar">
        <button
          v-for="(item, i) in ['计时器', '情书', '抽卡', '默契', '小猫', '愿望']"
          :key="i"
          :class="{ active: currentPage === i }"
          @click="goTo(i)"
        >{{ item }}</button>
      </nav>
      <div class="page-content">
        <LoveCounter v-if="currentPage === 0" />
        <LoveLetter v-if="currentPage === 1" />
        <LoveCard v-if="currentPage === 2" />
        <QuizGame v-if="currentPage === 3" />
        <CatGame v-if="currentPage === 4" />
        <WishList v-if="currentPage === 5" />
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
  display: flex;
  justify-content: center;
  gap: 4px;
  padding: 12px 8px;
  background: rgba(10, 10, 26, 0.9);
  backdrop-filter: blur(10px);
  z-index: 100;
  flex-wrap: wrap;
}

.nav-bar button {
  padding: 6px 14px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.7);
  font-size: 13px;
  transition: all 0.3s;
}

.nav-bar button.active {
  background: rgba(255, 107, 157, 0.3);
  color: #fff;
  box-shadow: 0 0 12px rgba(255, 107, 157, 0.3);
}

.nav-bar button:hover {
  background: rgba(255, 107, 157, 0.2);
  color: #fff;
}

.page-content {
  padding-top: 70px;
  min-height: 100vh;
}
</style>
