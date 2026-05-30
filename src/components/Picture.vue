<script setup>
import { ref, nextTick, onMounted, onUnmounted, computed } from 'vue'
import gsap from 'gsap'

// 同学表情包数据库 (增加了互动数据结构)
const rawPhotos = [
  { id: 1, url: 'https://s41.ax1x.com/2026/05/26/pmPnssI.jpg', caption: 'panda', lore: '高二运动会，那个著名的「我看你是懂了」表情。' },
  { id: 2, url: 'https://s41.ax1x.com/2026/05/26/pmPnrQA.jpg', caption: 'sexy', lore: '诚总课间不经意间的举动，被定格成为了永恒。' },
  { id: 3, url: 'https://s41.ax1x.com/2026/05/26/pmPuKmt.jpg', caption: '万豪', lore: '某次晚自习，他在后窗对上的那个眼神。' },
  { id: 4, url: 'https://s41.ax1x.com/2026/05/26/pmPuamq.jpg', caption: '晦气', lore: '经典「你最好有事」的眼神杀，常用于群聊开场。' },
  { id: 5, url: 'https://s41.ax1x.com/2026/05/26/pmPu61J.jpg', caption: '诚总', lore: '无需多言，懂的都懂。' },
  { id: 6, url: 'https://s41.ax1x.com/2026/05/26/pmPugXR.jpg',caption:'神秘人', lore: '档案已被加密，此人曾在高一元旦晚会一战成名。'}
]

// 响应式照片数据，用于存储本地互动
const photos = ref([])
const viewingPhoto = ref(null)
const viewingIndex = ref(-1)

// 锐评图标定义
const reactionIcons = {
  shock: { emoji: '😨', label: '黑历史' },
  funny: { emoji: '🤣', label: '神表情' },
  classic: { emoji: '👑', label: '经典' }
}

onMounted(() => {
  // 1. 初始化数据，尝试从本地读取互动记录
  const storedReactions = JSON.parse(localStorage.getItem('classmate_reactions') || '{}')
  photos.value = rawPhotos.map(p => ({
    ...p,
    reactions: storedReactions[p.id] || { shock: 0, funny: 0, classic: 0 }
  }))

  // 2. 进场交错动画
  gsap.from('.photo-card', {
    scale: 0.5,
    opacity: 0,
    duration: 0.8,
    stagger: 0.1,
    ease: 'back.out(1.5)',
    delay: 0.2
  })
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})

// 监听鼠标在卡片上的移动，实现 3D 倾斜效果 (可玩性加强)
function handleMouseMove(e, cardRef) {
  const rect = e.currentTarget.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  const centerX = rect.width / 2
  const centerY = rect.height / 2
  // 计算倾斜角度 (最大 10 度)
  const rotateX = ((y - centerY) / centerY) * -10
  const rotateY = ((x - centerX) / centerX) * 10

  gsap.to(e.currentTarget, {
    rotateX: rotateX,
    rotateY: rotateY,
    duration: 0.3,
    ease: 'power2.out',
    transformPerspective: 1000
  })
}

function handleMouseLeave(e) {
  gsap.to(e.currentTarget, {
    rotateX: 0,
    rotateY: 0,
    duration: 0.5,
    ease: 'elastic.out(1, 0.5)'
  })
}

// 键盘导航
function handleKeydown(e) {
  if (!viewingPhoto.value) return
  if (e.key === 'Escape') closePhoto()
  if (e.key === 'ArrowLeft') prevPhoto()
  if (e.key === 'ArrowRight') nextPhoto()
}

function openPhoto(photo, index) {
  viewingPhoto.value = photo
  viewingIndex.value = index
  nextTick(() => {
    gsap.fromTo('.lightbox', { opacity: 0 }, { opacity: 1, duration: 0.4 })
    gsap.fromTo('.lightbox-card',
        { scale: 0.7, rotation: -5, y: 50 },
        { scale: 1, rotation: 0, y: 0, duration: 0.6, ease: 'back.out(1.2)' }
    )
  })
}

function closePhoto() {
  gsap.to('.lightbox-card', {
    scale: 0.8, rotation: 5, opacity: 0, y: 50, duration: 0.4, ease: 'back.in(1.2)',
    onComplete: () => {
      viewingPhoto.value = null
      viewingIndex.value = -1
    }
  })
  gsap.to('.lightbox', { opacity: 0, duration: 0.3, delay: 0.1 })
}

// 随机召唤一位同学 (趣味交互)
function pickRandomClassmate() {
  closePhoto() // 先关闭当前的
  setTimeout(() => {
    const nextIdx = Math.floor(Math.random() * photos.value.length)
    openPhoto(photos.value[nextIdx], nextIdx)
  }, 100)
}

function prevPhoto() {
  if (viewingIndex.value > 0) {
    viewingIndex.value--
    viewingPhoto.value = photos.value[viewingIndex.value]
    gsap.fromTo('.lightbox-card', { x: -30, rotation: -3, opacity: 0 }, { x: 0, rotation: 0, opacity: 1, duration: 0.4, ease: 'power2.out' })
  }
}

function nextPhoto() {
  if (viewingIndex.value < photos.value.length - 1) {
    viewingIndex.value++
    viewingPhoto.value = photos.value[viewingIndex.value]
    gsap.fromTo('.lightbox-card', { x: 30, rotation: 3, opacity: 0 }, { x: 0, rotation: 0, opacity: 1, duration: 0.4, ease: 'power2.out' })
  }
}

// 锐评互动逻辑 (Persistent Data)
function addReaction(type) {
  if (!viewingPhoto.value) return
  // 增加计数
  viewingPhoto.value.reactions[type]++
  // 播放按钮弹跳动画
  gsap.fromTo(`.react-btn.${type}`, { scale: 0.8 }, { scale: 1, duration: 0.4, ease: 'elastic.out(1, 0.3)' })
  // 保存到本地
  saveReactions()
}

function saveReactions() {
  const reactionMap = {}
  photos.value.forEach(p => { reactionMap[p.id] = p.reactions })
  localStorage.setItem('classmate_reactions', JSON.stringify(reactionMap))
}
</script>

<template>
  <div class="picture-page">
    <div class="glow-orb orb-left"></div>
    <div class="glow-orb orb-right"></div>

    <div class="game-hud">
      <button class="hud-btn random-pick" @click="pickRandomClassmate" title="随机召唤一位同学">
        <span class="hud-icon">🪄</span>
      </button>
    </div>

    <div class="page-header">
      <h2 class="section-title">同学黑历史档案室</h2>
      <p class="subtitle">珍贵影像，严禁外传 | 已收录 {{ photos.length }} 份档案</p>
    </div>

    <div class="photo-grid">
      <div
          v-for="(photo, i) in photos"
          :key="i"
          class="photo-card glass-panel"
          @click="openPhoto(photo, i)"
          @mousemove="handleMouseMove($event)"
          @mouseleave="handleMouseLeave($event)"
      >
        <div class="photo-wrapper">
          <img :src="photo.url" :alt="photo.caption" loading="lazy" />
          <div class="photo-reactions-mini">
            <span v-if="photo.reactions.shock > 5">{{ reactionIcons.shock.emoji }}</span>
            <span v-if="photo.reactions.funny > 10">{{ reactionIcons.funny.emoji }}</span>
            <span v-if="photo.reactions.classic > 3">{{ reactionIcons.classic.emoji }}</span>
          </div>
        </div>
        <div class="photo-footer">
          <p class="photo-caption">{{ photo.caption }}</p>
          <span class="photo-view-lore">查看档案 ></span>
        </div>
      </div>
    </div>

    <teleport to="body">
      <div v-if="viewingPhoto" class="lightbox" @click.self="closePhoto">

        <button class="lightbox-btn lightbox-close" @click="closePhoto">×</button>
        <button v-if="viewingIndex > 0" class="lightbox-btn lightbox-nav prev" @click="prevPhoto">❮</button>
        <button v-if="viewingIndex < photos.length - 1" class="lightbox-btn lightbox-nav next" @click="nextPhoto">❯</button>

        <div class="lightbox-card glass-panel-dark">
          <div class="lightbox-img-wrapper">
            <img class="lightbox-img" :src="viewingPhoto.url" :alt="viewingPhoto.caption" />
          </div>

          <div class="lightbox-info">
            <h3 class="lightbox-caption">同学档案: {{ viewingPhoto.caption }}</h3>
            <p v-if="viewingPhoto.lore" class="lightbox-lore">{{ viewingPhoto.lore }}</p>

            <div class="reactions-panel">
              <button
                  v-for="(info, type) in reactionIcons"
                  :key="type"
                  class="react-btn"
                  :class="type"
                  @click="addReaction(type)"
                  :title="`锐评: ${info.label}`"
              >
                <span class="react-emoji">{{ info.emoji }}</span>
                <span class="react-count">{{ viewingPhoto.reactions[type] }}</span>
              </button>
            </div>

            <p class="lightbox-counter">档案索引: {{ viewingIndex + 1 }} / {{ photos.length }}</p>
          </div>
        </div>
      </div>
    </teleport>
  </div>
</template>

<style scoped>
.picture-page {
  padding: 40px 16px 80px; max-width: 900px; margin: 0 auto;
  position: relative; min-height: calc(100vh - 76px);
  /* 开启 3D 视差空间 */
  perspective: 1000px;
}

/* --- HUD 功能区 --- */
.game-hud { position: fixed; bottom: 20px; right: 20px; z-index: 100; display: flex; flex-direction: column; gap: 10px; }
.hud-btn {
  width: 50px; height: 50px; border-radius: 50%; border: none; cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  background: rgba(196, 77, 255, 0.2); backdrop-filter: blur(8px);
  border: 1px solid rgba(255,255,255,0.1); color: #fff;
  box-shadow: 0 5px 15px rgba(0,0,0,0.3); transition: all 0.3s;
}
.hud-btn:hover { transform: scale(1.1) rotate(15deg); background: rgba(196, 77, 255, 0.4); border-color: rgba(255, 107, 157, 0.5); }
.hud-icon { font-size: 24px; }

/* 氛围光晕 */
.glow-orb { position: absolute; border-radius: 50%; filter: blur(100px); opacity: 0.15; pointer-events: none; z-index: -1; }
.orb-left { width: 300px; height: 300px; background: #c44dff; top: 10%; left: -10%; }
.orb-right { width: 250px; height: 250px; background: #ff6b9d; bottom: 20%; right: -5%; }

.page-header { text-align: center; margin-bottom: 40px; }
.section-title {
  font-size: 2rem; font-weight: 800; margin: 0 0 8px;
  background: linear-gradient(135deg, #ffffff, #f0c4ff); -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.3));
}
.subtitle { color: rgba(255, 255, 255, 0.5); font-size: 0.9rem; margin: 0; letter-spacing: 1px; }

.glass-panel {
  background: rgba(255, 255, 255, 0.04); backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.08); border-radius: 16px;
}

/* 照片网格 */
.photo-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 18px; }

/* 拍立得 3D 卡片 */
.photo-card {
  padding: 10px 10px 15px; cursor: pointer;
  transition: box-shadow 0.3s; /* 旋转由 JS 接管，基础 transition 只负责发光 */
  display: flex; flex-direction: column;
  /* 确保 3D 转换子元素可见 */
  transform-style: preserve-3d;
}

.photo-card:hover {
  border-color: rgba(255, 107, 157, 0.3);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3), 0 0 20px rgba(255, 107, 157, 0.1);
  z-index: 2;
}

.photo-wrapper {
  width: 100%; aspect-ratio: 1; border-radius: 8px; overflow: hidden; position: relative;
  background: rgba(0, 0, 0, 0.2);
  /* 3D z-axis lift */
  transform: translateZ(20px);
}
.photo-wrapper img { width: 100%; height: 100%; object-fit: cover; }

/* 迷你锐评统计 (可玩性显示) */
.photo-reactions-mini {
  position: absolute; bottom: 5px; right: 5px; display: flex; gap: 3px; font-size: 12px;
  background: rgba(0,0,0,0.5); padding: 2px 5px; border-radius: 10px; backdrop-filter: blur(2px);
}

.photo-footer { margin-top: 10px; display: flex; flex-direction: column; align-items: center; gap: 3px; transform: translateZ(15px);}
.photo-caption { margin: 0; font-size: 0.9rem; color: #fff; font-weight: 600; text-align: center; }
.photo-view-lore { font-size: 0.7rem; color: rgba(255, 255, 255, 0.4); }
.photo-card:hover .photo-view-lore { color: #ff9a9e; }

/* === 沉浸式大图画廊 (档案模式) === */
.lightbox {
  position: fixed; inset: 0; z-index: 9999;
  background: rgba(5, 2, 10, 0.88); display: flex; align-items: center; justify-content: center;
  backdrop-filter: blur(15px); -webkit-backdrop-filter: blur(15px);
}

.lightbox-btn {
  position: absolute; display: flex; align-items: center; justify-content: center;
  background: rgba(255, 255, 255, 0.05); border: 1px solid rgba(255, 255, 255, 0.1); color: #fff;
  border-radius: 50%; cursor: pointer; transition: all 0.3s; z-index: 10000;
}
.lightbox-btn:hover { background: rgba(255, 107, 157, 0.2); border-color: rgba(255, 107, 157, 0.4); transform: scale(1.1); }

.lightbox-close { top: 20px; right: 20px; width: 40px; height: 40px; font-size: 24px; }
.lightbox-nav { top: 50%; margin-top: -30px; width: 60px; height: 60px; font-size: 24px; }
.lightbox-nav.prev { left: 24px; }
.lightbox-nav.next { right: 24px; }

/* 强化的毛玻璃大卡片 */
.lightbox-card {
  padding: 15px 15px 30px; max-width: 90vw; max-height: 90vh;
  border-radius: 20px; text-align: center; display: flex; flex-direction: column; align-items: center;
  box-shadow: 0 30px 70px rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255,255,255,0.06);
}
.glass-panel-dark { background: rgba(10, 10, 15, 0.6); backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px); }

.lightbox-img-wrapper {
  width: 100%; max-height: 60vh; border-radius: 12px; overflow: hidden;
  margin-bottom: 20px; box-shadow: inset 0 0 20px rgba(0,0,0,0.5);
}
.lightbox-img { max-width: 100%; max-height: 60vh; object-fit: contain; }

.lightbox-info { width: 90%; max-width: 500px; }
.lightbox-caption { color: #fff; font-size: 1.25rem; font-weight: 700; margin: 0 0 8px 0; }
.lightbox-lore { color: rgba(255, 255, 255, 0.6); font-size: 0.9rem; line-height: 1.6; margin: 0 0 25px 0; background: rgba(0,0,0,0.2); padding: 10px; border-radius: 8px; text-align: left;}

/* 锐评系统 UI */
.reactions-panel { display: flex; justify-content: center; gap: 15px; margin-bottom: 20px; }
.react-btn {
  display: flex; align-items: center; gap: 8px; padding: 8px 18px; border-radius: 20px;
  border: 1px solid rgba(255,255,255,0.1); background: rgba(255,255,255,0.03);
  cursor: pointer; color: #fff; transition: all 0.3s;
}
.react-btn:hover { transform: translateY(-3px) scale(1.05); background: rgba(255,255,255,0.08); }
.react-btn.shock:hover { border-color: #ff9800; background: rgba(255, 152, 0, 0.1); }
.react-btn.funny:hover { border-color: #ffd700; background: rgba(255, 215, 0, 0.1); }
.react-btn.classic:hover { border-color: #ff6b9d; background: rgba(255, 107, 157, 0.1); }
.react-emoji { font-size: 20px; }
.react-count { font-weight: 600; font-size: 0.9rem; min-width: 20px; text-align: left; }

.lightbox-counter { color: rgba(255, 255, 255, 0.3); font-size: 0.8rem; margin: 0; font-weight: 600; letter-spacing: 1px; }

/* 移动端适配 */
@media (max-width: 768px) {
  .photo-grid { grid-template-columns: repeat(2, 1fr); gap: 12px; }
  .photo-card { padding: 8px 8px 12px; }
  .photo-caption { font-size: 0.8rem; }
  .game-hud { bottom: 15px; right: 15px; }
  .hud-btn { width: 40px; height: 40px; }
  .hud-icon { font-size: 20px; }
  .lightbox-btn { width: 44px; height: 44px; background: rgba(0,0,0,0.5); }
  .lightbox-nav.prev { left: 10px; }
  .lightbox-nav.next { right: 10px; }
  .lightbox-card { padding: 10px 10px 20px; }
  .lightbox-img { max-height: 50vh; }
  .reactions-panel { gap: 8px; flex-wrap: wrap; }
  .react-btn { padding: 6px 12px; font-size: 0.8rem; }
}
</style>