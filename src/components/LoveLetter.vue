<script setup>
import { ref, onMounted } from 'vue'
import gsap from 'gsap'

const lines = [
  '亲爱的，',
  '',
  '其实我一直觉得，遇见你是一件很幸运的事情。',
  '',
  '虽然我们没有很多照片，',
  '但我们有很多只有彼此才懂的小瞬间。',
  '',
  '那些一起熬过的夜晚，',
  '那些突然发来的消息，',
  '那些说不出口的想念，',
  '都是我最珍贵的回忆。',
  '',
  '谢谢你出现在我的生命里。',
  '未来的日子，我们一起走。',
  '',
  '永远喜欢你的人'
]

const visibleLines = ref(0)
const opened = ref(false)

function openLetter() {
  opened.value = true
  let i = 0
  const timer = setInterval(() => {
    i++
    visibleLines.value = i
    if (i >= lines.length) clearInterval(timer)
  }, 400)
}

onMounted(() => {
  gsap.from('.letter-envelope', { opacity: 0, scale: 0.9, duration: 0.8 })
})
</script>

<template>
  <div class="love-letter">
    <h2 class="section-title">我想对你说</h2>
    <div v-if="!opened" class="letter-envelope" @click="openLetter">
      <div class="envelope-body">
        <div class="envelope-flap"></div>
        <div class="envelope-heart">💌</div>
        <p class="envelope-hint">点击打开这封信</p>
      </div>
    </div>
    <div v-else class="letter-paper">
      <div class="paper-content">
        <p
          v-for="(line, i) in lines"
          :key="i"
          class="letter-line"
          :class="{ visible: i < visibleLines, empty: line === '' }"
        >{{ line }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.love-letter {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 70px);
  padding: 40px 20px;
}

.section-title {
  font-size: 1.5rem;
  margin-bottom: 40px;
  color: rgba(255, 255, 255, 0.9);
}

.letter-envelope {
  cursor: pointer;
  transition: transform 0.3s;
}

.letter-envelope:hover {
  transform: scale(1.05);
}

.envelope-body {
  width: 240px;
  height: 160px;
  background: linear-gradient(135deg, #2a1f3d, #1a1230);
  border: 1px solid rgba(255, 107, 157, 0.3);
  border-radius: 12px;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.envelope-flap {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 50%;
  background: linear-gradient(180deg, rgba(255, 107, 157, 0.1), transparent);
  border-radius: 12px 12px 0 0;
  clip-path: polygon(0 0, 50% 60%, 100% 0);
}

.envelope-heart {
  font-size: 40px;
  margin-bottom: 8px;
}

.envelope-hint {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.5);
}

.letter-paper {
  max-width: 500px;
  width: 100%;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 16px;
  padding: 40px 32px;
  backdrop-filter: blur(10px);
}

.letter-line {
  font-size: 0.95rem;
  line-height: 2;
  color: rgba(255, 255, 255, 0.85);
  opacity: 0;
  transform: translateY(10px);
  transition: all 0.5s ease;
}

.letter-line.visible {
  opacity: 1;
  transform: translateY(0);
}

.letter-line.empty {
  height: 16px;
}
</style>
