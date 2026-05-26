<script setup>
import { ref, nextTick } from 'vue'
import gsap from 'gsap'

const photos = [
   { url: 'https://s41.ax1x.com/2026/05/26/pmPnssI.jpg', caption: 'panda' },
   { url: 'https://s41.ax1x.com/2026/05/26/pmPnrQA.jpg', caption: 'sexy' },
   { url: 'https://s41.ax1x.com/2026/05/26/pmPuKmt.jpg', caption: '万豪' },
   { url: 'https://s41.ax1x.com/2026/05/26/pmPuamq.jpg', caption: '晦气' },
   { url: 'https://s41.ax1x.com/2026/05/26/pmPu61J.jpg', caption: '诚总' },
   { url: 'https://s41.ax1x.com/2026/05/26/pmPugXR.jpg',caption:'神秘人'}
]

const viewingPhoto = ref(null)
const viewingIndex = ref(-1)

function openPhoto(photo, index) {
  viewingPhoto.value = photo
  viewingIndex.value = index
  nextTick(() => {
    gsap.fromTo('.lightbox', { opacity: 0 }, { opacity: 1, duration: 0.3 })
    gsap.fromTo('.lightbox-img', { scale: 0.85, opacity: 0 }, { scale: 1, opacity: 1, duration: 0.4, ease: 'back.out(1.4)' })
  })
}

function closePhoto() {
  gsap.to('.lightbox', {
    opacity: 0,
    duration: 0.25,
    onComplete: () => {
      viewingPhoto.value = null
      viewingIndex.value = -1
    }
  })
}

function prevPhoto() {
  if (viewingIndex.value > 0) {
    viewingIndex.value--
    viewingPhoto.value = photos[viewingIndex.value]
    gsap.fromTo('.lightbox-img', { x: -30, opacity: 0 }, { x: 0, opacity: 1, duration: 0.3 })
  }
}

function nextPhoto() {
  if (viewingIndex.value < photos.length - 1) {
    viewingIndex.value++
    viewingPhoto.value = photos[viewingIndex.value]
    gsap.fromTo('.lightbox-img', { x: 30, opacity: 0 }, { x: 0, opacity: 1, duration: 0.3 })
  }
}
</script>

<template>
  <div class="picture-page">
    <div class="page-header">
      <h2>📷 我们的照片墙</h2>
      <p class="subtitle">每一张都是珍贵的回忆</p>
    </div>

    <div v-if="photos.length === 0" class="empty-state">
      <span class="empty-icon">🖼️</span>
      <p>还没有照片哦～</p>
      <p class="empty-hint">在代码里添加图片链接就能看到啦</p>
    </div>

    <div v-else class="photo-grid">
      <div
        v-for="(photo, i) in photos"
        :key="i"
        class="photo-card"
        @click="openPhoto(photo, i)"
      >
        <div class="photo-wrapper">
          <img :src="photo.url" :alt="photo.caption" loading="lazy" />
        </div>
        <p v-if="photo.caption" class="photo-caption">{{ photo.caption }}</p>
      </div>
    </div>

    <teleport to="body">
      <div v-if="viewingPhoto" class="lightbox" @click.self="closePhoto">
        <button class="lightbox-close" @click="closePhoto">✕</button>
        <button v-if="viewingIndex > 0" class="lightbox-nav prev" @click="prevPhoto">‹</button>
        <button v-if="viewingIndex < photos.length - 1" class="lightbox-nav next" @click="nextPhoto">›</button>
        <div class="lightbox-content">
          <img class="lightbox-img" :src="viewingPhoto.url" :alt="viewingPhoto.caption" />
          <p v-if="viewingPhoto.caption" class="lightbox-caption">{{ viewingPhoto.caption }}</p>
          <p class="lightbox-counter">{{ viewingIndex + 1 }} / {{ photos.length }}</p>
        </div>
      </div>
    </teleport>
  </div>
</template>

<style scoped>
.picture-page {
  padding: 20px 16px 40px;
  max-width: 700px;
  margin: 0 auto;
}

.page-header {
  text-align: center;
  margin-bottom: 28px;
}

.page-header h2 {
  font-size: 22px;
  color: #fff;
  margin: 0 0 6px;
}

.subtitle {
  color: rgba(255, 255, 255, 0.5);
  font-size: 13px;
  margin: 0;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: rgba(255, 255, 255, 0.5);
}

.empty-icon {
  font-size: 48px;
  display: block;
  margin-bottom: 12px;
}

.empty-state p {
  margin: 4px 0;
  font-size: 14px;
}

.empty-hint {
  font-size: 12px !important;
  color: rgba(255, 255, 255, 0.3);
}

.photo-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.photo-card {
  border-radius: 12px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  cursor: pointer;
  transition: all 0.3s ease;
}

.photo-card:hover {
  transform: translateY(-2px);
  border-color: rgba(255, 107, 157, 0.3);
  box-shadow: 0 4px 20px rgba(255, 107, 157, 0.15);
}

.photo-wrapper {
  width: 100%;
  aspect-ratio: 1;
  overflow: hidden;
}

.photo-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.photo-card:hover .photo-wrapper img {
  transform: scale(1.05);
}

.photo-caption {
  padding: 8px 10px;
  margin: 0;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.7);
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.lightbox {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.92);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(10px);
}

.lightbox-close {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: #fff;
  font-size: 18px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lightbox-nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: #fff;
  font-size: 24px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lightbox-nav.prev { left: 16px; }
.lightbox-nav.next { right: 16px; }

.lightbox-content {
  text-align: center;
  max-width: 90vw;
  max-height: 85vh;
}

.lightbox-img {
  max-width: 90vw;
  max-height: 70vh;
  border-radius: 8px;
  object-fit: contain;
}

.lightbox-caption {
  color: rgba(255, 255, 255, 0.8);
  font-size: 14px;
  margin: 12px 0 4px;
}

.lightbox-counter {
  color: rgba(255, 255, 255, 0.4);
  font-size: 12px;
  margin: 4px 0 0;
}
</style>
