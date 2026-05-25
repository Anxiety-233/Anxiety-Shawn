<script setup>
import { onMounted, ref, onUnmounted } from 'vue'

const canvas = ref(null)
let animId

onMounted(() => {
  const cvs = canvas.value
  const ctx = cvs.getContext('2d')
  let w = cvs.width = window.innerWidth
  let h = cvs.height = window.innerHeight

  const stars = Array.from({ length: 120 }, () => ({
    x: Math.random() * w,
    y: Math.random() * h,
    r: Math.random() * 1.2 + 0.3,
    alpha: Math.random(),
    speed: Math.random() * 0.008 + 0.002
  }))

  function animate() {
    ctx.clearRect(0, 0, w, h)
    stars.forEach(s => {
      s.alpha += s.speed
      if (s.alpha > 1 || s.alpha < 0.1) s.speed *= -1
      ctx.beginPath()
      ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(255, 255, 255, ${s.alpha * 0.6})`
      ctx.fill()
    })
    animId = requestAnimationFrame(animate)
  }
  animate()

  const onResize = () => {
    w = cvs.width = window.innerWidth
    h = cvs.height = window.innerHeight
    stars.forEach(s => { s.x = Math.random() * w; s.y = Math.random() * h })
  }
  window.addEventListener('resize', onResize)
})

onUnmounted(() => { cancelAnimationFrame(animId) })
</script>

<template>
  <canvas ref="canvas" class="star-bg"></canvas>
</template>

<style scoped>
.star-bg {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  pointer-events: none;
}
</style>
