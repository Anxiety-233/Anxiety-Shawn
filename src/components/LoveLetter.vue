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
const envelopeHover = ref(false)

function openLetter() {
  opened.value = true
  gsap.from('.letter-paper', { scale: 0.9, opacity: 0, duration: 0.6, ease: 'back.out(1.5)' })
  let i = 0
  const timer = setInterval(() => {
    i++
    visibleLines.value = i
    if (i >= lines.length) clearInterval(timer)
  }, 350)
}

onMounted(() => {
  gsap.from('.letter-envelope', { opacity: 0, y: 30, duration: 0.8, ease: 'power3.out' })
})
</script>

<template>
  <div class="love-letter">
    <div class="glow-orb orb-pink"></div>

    <h2 class="section-title">我想对你说</h2>
    <p class="section-sub">一封写给你的信</p>

    <div
      v-if="!opened"
      class="letter-envelope"
      @click="openLetter"
      @mouseenter="envelopeHover = true"
      @mouseleave="envelopeHover = false"
    >
      <div class="envelope-outer">
        <div class="envelope-flap" :class="{ hover: envelopeHover }"></div>
        <div class="envelope-body">
          <div class="envelope-heart">💌</div>
          <div class="envelope-seal"></div>
        </div>
      </div>
      <p class="envelope-hint">点击打开这封信</p>
    </div>

    <div v-else class="letter-paper">
      <div class="paper-decoration top-left">✦</div>
      <div class="paper-decoration top-right">✦</div>
      <div class="paper-decoration bottom-left">✦</div>
      <div class="paper-decoration bottom-right">✦</div>
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
  min-height: calc(100vh - 76px);
  padding: 40px 20px;
  position: relative;
}

.glow-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(100px);
  opacity: 0.12;
  pointer-events: none;
}

.orb-pink {
  width: 400px;
  height: 400px;
  background: #c44dff;
  top: 20%;
  right: -15%;
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
  margin-bottom: 48px;
}

.letter-envelope {
  cursor: pointer;
  text-align: center;
}

.envelope-outer {
  width: 260px;
  height: 180px;
  position: relative;
  margin: 0 auto 16px;
  transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.letter-envelope:hover .envelope-outer {
  transform: translateY(-8px) scale(1.02);
}

.envelope-body {
  width: 100%;
  height: 100%;
  background: linear-gradient(145deg, #1e1535, #150f28);
  border: 1px solid rgba(255, 107, 157, 0.2);
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3), inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

.envelope-flap {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 55%;
  background: linear-gradient(180deg, rgba(255, 107, 157, 0.08), transparent);
  border-radius: 16px 16px 0 0;
  clip-path: polygon(0 0, 50% 65%, 100% 0);
  transition: transform 0.4s ease;
  transform-origin: top center;
}

.envelope-flap.hover {
  transform: rotateX(-20deg);
}

.envelope-heart {
  font-size: 48px;
  filter: drop-shadow(0 4px 12px rgba(255, 107, 157, 0.3));
  animation: gentleFloat 3s ease-in-out infinite;
}

@keyframes gentleFloat {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}

.envelope-seal {
  position: absolute;
  bottom: 16px;
  right: 16px;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  opacity: 0.6;
}

.envelope-hint {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.35);
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 0.35; }
  50% { opacity: 0.7; }
}

.letter-paper {
  max-width: 480px;
  width: 100%;
  background: linear-gradient(145deg, rgba(255, 255, 255, 0.04), rgba(255, 255, 255, 0.02));
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 20px;
  padding: 48px 36px;
  backdrop-filter: blur(10px);
  position: relative;
  box-shadow: 0 16px 48px rgba(0, 0, 0, 0.2);
}

.paper-decoration {
  position: absolute;
  font-size: 12px;
  color: rgba(255, 107, 157, 0.3);
}

.top-left { top: 16px; left: 16px; }
.top-right { top: 16px; right: 16px; }
.bottom-left { bottom: 16px; left: 16px; }
.bottom-right { bottom: 16px; right: 16px; }

.letter-line {
  font-size: 0.95rem;
  line-height: 2.2;
  color: rgba(255, 255, 255, 0.85);
  opacity: 0;
  transform: translateY(8px);
  transition: all 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.letter-line.visible {
  opacity: 1;
  transform: translateY(0);
}

.letter-line.empty {
  height: 20px;
}
</style>
