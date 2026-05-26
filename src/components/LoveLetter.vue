<script setup>
import { ref, onMounted, nextTick } from 'vue'
import gsap from 'gsap'

// 💌 剧本库：可以随时添加你们的新信件
const letters = [
  {
    title: '致 我们的初见',
    theme: 'pink',
    lines: [
      '亲爱的，',
      '',
      '其实我一直觉得，遇见你是一件很幸运的事情。',
      '还记得我们刚认识的时候吗？',
      '那时候总有说不完的话，连晚安都要说好几遍。',
      '',
      '虽然现在没有了当初的客气，',
      '但我们有了只有彼此才懂的默契和小瞬间。',
      '',
      '谢谢你出现在我的生命里。',
      '未来的日子，我们也要一起慢慢走。',
      '',
      '永远爱你的专属小笨蛋'
    ]
  },
  {
    title: '致 柴米油盐的日常',
    theme: 'orange',
    lines: [
      '宝宝，',
      '',
      '今天也是好想你的一天。',
      '最近生活虽然平淡，但只要想到你在，就觉得很心安。',
      '',
      '喜欢和你一起窝在沙发上看剧，',
      '喜欢和你为了晚上吃什么而纠结半天，',
      '喜欢你总是不经意间给我的小惊喜。',
      '',
      '那些再普通不过的日子，',
      '因为有了你的参与，全都闪闪发光。',
      '',
      'Mua~😘'
    ]
  },
  {
    title: '致 遥远的未来',
    theme: 'purple',
    lines: [
      '我最爱的人，',
      '',
      '不知道当你看到这封信的时候，我们正在做什么？',
      '也许正在为了旅游做攻略，',
      '也许正在属于我们的小家里打扫卫生。',
      '',
      '我想象过无数个未来的画面，',
      '每一个画面里，都有你。',
      '',
      '即使以后老了，走不动了，',
      '我也要牵着你的手，给你念年轻时写的情书。',
      '',
      '执子之手，与子偕老。'
    ]
  }
]

const currentIndex = ref(0)
const status = ref('sealed') // sealed -> opening -> reading -> stamped
const visibleLines = ref(0)
const particles = ref([])
let heartId = 0

// 切换信件
function nextLetter() {
  if (status.value !== 'sealed') return
  currentIndex.value = (currentIndex.value + 1) % letters.length
}
function prevLetter() {
  if (status.value !== 'sealed') return
  currentIndex.value = (currentIndex.value - 1 + letters.length) % letters.length
}

// 拆开信封的 GSAP 动画序列
function openEnvelope() {
  if (status.value !== 'sealed') return
  status.value = 'opening'

  const tl = gsap.timeline({
    onComplete: () => {
      status.value = 'reading'
      startTyping()
    }
  })

  // 1. 火漆印章消失
  tl.to('.wax-seal', { scale: 0, opacity: 0, duration: 0.3, ease: 'back.in(2)' })

  // 2. 信封盖翻开 (3D效果)
  tl.to('.envelope-flap', { rotateX: 180, duration: 0.6, ease: 'power2.inOut' })

  // 3. 信纸抽出
  tl.to('.envelope-paper-preview', { y: -80, duration: 0.6, ease: 'power2.out' }, '-=0.2')

  // 4. 信封整体淡出，信纸全屏放大呈现
  tl.to('.envelope-container', { scale: 0.8, opacity: 0, duration: 0.5, ease: 'power2.in' })
}

// 逐行打字机效果
function startTyping() {
  visibleLines.value = 0
  const currentLines = letters[currentIndex.value].lines
  let i = 0

  nextTick(() => {
    gsap.fromTo('.letter-paper',
        { y: 50, opacity: 0, scale: 0.95 },
        { y: 0, opacity: 1, scale: 1, duration: 0.8, ease: 'back.out(1.2)' }
    )

    const timer = setInterval(() => {
      i++
      visibleLines.value = i
      if (i >= currentLines.length) {
        clearInterval(timer)
      }
    }, 450) // 每行显示的间隔时间
  })
}

// 盖上心印并触发粒子爆炸
function stampLetter() {
  if (status.value === 'stamped') return
  status.value = 'stamped'

  gsap.fromTo('.heart-stamp',
      { scale: 3, opacity: 0, rotation: -20 },
      { scale: 1, opacity: 1, rotation: -5, duration: 0.5, ease: 'bounce.out' }
  )

  // 粒子爆炸
  for (let i = 0; i < 20; i++) {
    const id = heartId++
    particles.value.push({
      id,
      x: (Math.random() - 0.5) * 200,
      y: (Math.random() - 0.5) * 200 - 100,
      size: Math.random() * 15 + 10,
      delay: Math.random() * 0.2
    })
    setTimeout(() => {
      particles.value = particles.value.filter(p => p.id !== id)
    }, 2000)
  }
}

// 重置回信封状态
function resetLetter() {
  status.value = 'sealed'
  visibleLines.value = 0
  gsap.set('.envelope-flap', { rotateX: 0 })
  gsap.set('.wax-seal', { scale: 1, opacity: 1 })
  gsap.set('.envelope-paper-preview', { y: 0 })
  gsap.set('.envelope-container', { scale: 1, opacity: 1 })
}
</script>

<template>
  <div class="love-letter" :class="`theme-${letters[currentIndex].theme}`">
    <div class="glow-orb orb-1"></div>
    <div class="glow-orb orb-2"></div>

    <div class="header" v-if="status === 'sealed' || status === 'opening'">
      <h2 class="section-title">专属情书</h2>
      <p class="section-sub">选择一封你想拆开的信件</p>
    </div>

    <div class="envelope-stage" v-show="status === 'sealed' || status === 'opening'">
      <button class="nav-btn prev" @click="prevLetter" v-if="status === 'sealed'">❮</button>

      <div class="envelope-container" @click="openEnvelope">
        <div class="envelope-title">{{ letters[currentIndex].title }}</div>

        <div class="envelope-body">
          <div class="envelope-back"></div>
          <div class="envelope-paper-preview"></div>
          <div class="envelope-front"></div>

          <div class="envelope-flap">
            <div class="wax-seal">
              <span class="seal-icon">💝</span>
            </div>
          </div>
        </div>

        <div class="click-hint" v-if="status === 'sealed'">点击火漆印章拆信</div>
      </div>

      <button class="nav-btn next" @click="nextLetter" v-if="status === 'sealed'">❯</button>
    </div>

    <div class="reading-stage" v-if="status === 'reading' || status === 'stamped'">
      <div class="letter-paper">
        <div class="paper-texture"></div>
        <div class="paper-corner top-left">✦</div>
        <div class="paper-corner top-right">✦</div>
        <div class="paper-corner bottom-left">✦</div>
        <div class="paper-corner bottom-right">✦</div>

        <div class="paper-content">
          <p
              v-for="(line, i) in letters[currentIndex].lines"
              :key="i"
              class="letter-line"
              :class="{ visible: i < visibleLines, empty: line === '' }"
          >{{ line }}</p>
        </div>

        <div v-if="visibleLines >= letters[currentIndex].lines.length" class="stamp-section">
          <div v-if="status === 'stamped'" class="heart-stamp">
            <span class="stamp-border">Accepted</span>
            <span class="stamp-emoji">💋</span>
          </div>
          <button v-else class="stamp-btn" @click="stampLetter">盖上专属心印</button>
        </div>
      </div>

      <button v-if="status === 'stamped'" class="back-btn" @click="resetLetter">把信收好</button>
    </div>

    <div class="particle-container">
      <div
          v-for="p in particles"
          :key="p.id"
          class="particle-heart"
          :style="{
          '--tx': `${p.x}px`,
          '--ty': `${p.y}px`,
          fontSize: `${p.size}px`,
          animationDelay: `${p.delay}s`
        }"
      >❤️</div>
    </div>
  </div>
</template>

<style scoped>
.love-letter {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: calc(100vh - 76px);
  padding: 40px 15px 80px;
  position: relative;
  overflow: hidden;
  background: radial-gradient(circle at top, #1a1525, #0a0a10);
  font-family: 'PingFang SC', serif;
  transition: all 0.8s ease;
}

/* 动态主题氛围光 */
.glow-orb { position: absolute; border-radius: 50%; filter: blur(90px); opacity: 0.15; pointer-events: none; transition: background 1s ease; }
.theme-pink .orb-1 { background: #ff6b9d; }
.theme-orange .orb-1 { background: #ff9800; }
.theme-purple .orb-1 { background: #ab47bc; }
.orb-1 { width: 350px; height: 350px; top: -50px; right: -100px; }
.orb-2 { width: 250px; height: 250px; bottom: 10%; left: -50px; background: #42a5f5; opacity: 0.1; }

.header { text-align: center; margin-bottom: 30px; z-index: 10; }
.section-title { font-size: 1.8rem; font-weight: 800; margin-bottom: 8px; background: linear-gradient(135deg, #fff, #f0c4ff); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.section-sub { font-size: 0.85rem; color: rgba(255, 255, 255, 0.4); }

/* === 💌 信封阶段 CSS === */
.envelope-stage { display: flex; align-items: center; justify-content: center; gap: 15px; width: 100%; max-width: 500px; margin-top: 40px; z-index: 10; perspective: 1000px; }

.nav-btn {
  background: rgba(255, 255, 255, 0.05); border: 1px solid rgba(255, 255, 255, 0.1);
  color: #fff; width: 40px; height: 40px; border-radius: 50%; font-size: 1.2rem;
  cursor: pointer; transition: all 0.3s; backdrop-filter: blur(4px);
}
.nav-btn:hover { background: rgba(255, 255, 255, 0.15); transform: scale(1.1); }

.envelope-container { cursor: pointer; text-align: center; position: relative; width: 260px; }
.envelope-title { font-size: 1.1rem; color: #fff; font-weight: 600; margin-bottom: 25px; letter-spacing: 1px; text-shadow: 0 2px 10px rgba(0,0,0,0.5); }

.envelope-body {
  width: 260px; height: 160px; position: relative; margin: 0 auto;
  transform-style: preserve-3d; transition: transform 0.4s;
}
.envelope-container:hover .envelope-body { transform: translateY(-5px); }

/* 信封各层级结构 (纯CSS绘制) */
.envelope-back { position: absolute; inset: 0; background: linear-gradient(145deg, #2a203a, #1a1525); border-radius: 8px; box-shadow: 0 15px 35px rgba(0,0,0,0.4); z-index: 1; }
.envelope-paper-preview { position: absolute; left: 10px; right: 10px; top: 10px; height: 140px; background: #fff; border-radius: 6px; z-index: 2; box-shadow: 0 -2px 10px rgba(0,0,0,0.1); }
.envelope-front {
  position: absolute; inset: 0; z-index: 3; border-radius: 8px;
  background: linear-gradient(145deg, #322545, #241a32);
  clip-path: polygon(0 0, 100% 0, 100% 100%, 50% 65%, 0 100%);
  border: 1px solid rgba(255,255,255,0.05);
}
.envelope-flap {
  position: absolute; top: 0; left: 0; right: 0; height: 60%;
  background: linear-gradient(180deg, #3a2b4f, #2a203a);
  clip-path: polygon(0 0, 100% 0, 50% 100%);
  transform-origin: top; z-index: 4; border-radius: 8px 8px 0 0;
}

.wax-seal {
  position: absolute; bottom: 10px; left: 50%; margin-left: -20px;
  width: 40px; height: 40px; border-radius: 50%;
  background: radial-gradient(circle at 30% 30%, #ff4b4b, #b30000);
  box-shadow: 0 4px 10px rgba(0,0,0,0.5), inset 0 2px 5px rgba(255,255,255,0.3);
  display: flex; align-items: center; justify-content: center; z-index: 5;
}
.seal-icon { font-size: 16px; filter: drop-shadow(0 1px 2px rgba(0,0,0,0.5)); animation: pulseHeart 2s infinite; }

.click-hint { margin-top: 30px; font-size: 0.8rem; color: rgba(255, 255, 255, 0.4); animation: pulse 2s infinite; }

/* === 📜 阅读阶段 CSS === */
.reading-stage { width: 100%; display: flex; flex-direction: column; align-items: center; z-index: 20; }

.letter-paper {
  max-width: 500px; width: 90%; background: #faf9f6; /* 米白信纸色 */
  border-radius: 12px; padding: 50px 35px 60px; position: relative;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.4), inset 0 0 40px rgba(0,0,0,0.02);
  color: #333; overflow: hidden;
}

.paper-texture { position: absolute; inset: 0; opacity: 0.4; background-image: radial-gradient(#d5d0c8 1px, transparent 1px); background-size: 20px 20px; pointer-events: none; }
.paper-corner { position: absolute; font-size: 14px; color: #d0c5b5; }
.top-left { top: 20px; left: 20px; } .top-right { top: 20px; right: 20px; }
.bottom-left { bottom: 20px; left: 20px; } .bottom-right { bottom: 20px; right: 20px; }

.paper-content { position: relative; z-index: 2; }
.letter-line {
  font-size: 1.05rem; line-height: 2.2; color: #4a4440;
  opacity: 0; transform: translateY(10px) scale(0.98); filter: blur(2px);
  transition: all 0.8s cubic-bezier(0.25, 0.8, 0.25, 1);
}
.letter-line.visible { opacity: 1; transform: translateY(0) scale(1); filter: blur(0); }
.letter-line.empty { height: 24px; }

/* 互动印章区 */
.stamp-section { margin-top: 40px; display: flex; justify-content: flex-end; position: relative; min-height: 80px; }
.stamp-btn {
  background: transparent; border: 1px dashed #d0c5b5; color: #a59b8c;
  padding: 10px 20px; border-radius: 8px; cursor: pointer; transition: all 0.3s;
  font-family: inherit; font-size: 0.9rem; position: relative; z-index: 3;
}
.stamp-btn:hover { border-color: #ff6b9d; color: #ff6b9d; background: rgba(255, 107, 157, 0.05); }

.heart-stamp {
  position: absolute; right: 10px; top: -10px; width: 100px; height: 100px;
  border: 4px solid rgba(220, 20, 60, 0.6); border-radius: 50%;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  color: rgba(220, 20, 60, 0.8); z-index: 4; transform: rotate(-15deg);
  box-shadow: inset 0 0 10px rgba(220,20,60,0.2);
}
.stamp-border { font-size: 0.7rem; font-weight: bold; letter-spacing: 2px; border-bottom: 1px solid rgba(220,20,60,0.5); padding-bottom: 2px; }
.stamp-emoji { font-size: 32px; margin-top: 5px; opacity: 0.9; }

.back-btn { margin-top: 30px; background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2); color: #fff; padding: 12px 30px; border-radius: 25px; cursor: pointer; backdrop-filter: blur(5px); transition: all 0.3s; }
.back-btn:hover { background: rgba(255,255,255,0.2); }

/* 动画定义 */
@keyframes pulse { 0%, 100% { opacity: 0.3; } 50% { opacity: 0.8; } }
@keyframes pulseHeart { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.15); } }

/* 粒子特效 */
.particle-container { position: fixed; inset: 0; pointer-events: none; z-index: 100; display: flex; align-items: center; justify-content: center; }
.particle-heart { position: absolute; animation: explode 2s cubic-bezier(0.1, 0.8, 0.3, 1) forwards; opacity: 0; filter: drop-shadow(0 0 5px rgba(255, 107, 157, 0.8)); }
@keyframes explode {
  0% { transform: translate(0, 0) scale(0) rotate(0deg); opacity: 1; }
  100% { transform: translate(var(--tx), var(--ty)) scale(1) rotate(180deg); opacity: 0; }
}
</style>