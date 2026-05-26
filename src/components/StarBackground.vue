<script setup>
import { onMounted, ref, onUnmounted } from 'vue'

const canvas = ref(null)
let animId

// 鼠标位置用于视差效果
const mousePos = ref({ x: 0, y: 0 })

onMounted(() => {
  const cvs = canvas.value
  const ctx = cvs.getContext('2d')
  let w = cvs.width = window.innerWidth
  let h = cvs.height = window.innerHeight

  const stars = Array.from({ length: 150 }, () => ({
    x: Math.random() * w,
    y: Math.random() * h,
    r: Math.random() * 1.5 + 0.5,
    alpha: Math.random(),
    speed: Math.random() * 0.005 + 0.001,
    z: Math.random() * 0.5 + 0.1 // 深度系数
  }))

  const handleMouseMove = (e) => {
    mousePos.value = { x: (e.clientX / window.innerWidth) - 0.5, y: (e.clientY / window.innerHeight) - 0.5 }
  }
  window.addEventListener('mousemove', handleMouseMove)

  function animate() {
    ctx.clearRect(0, 0, w, h)

    // 背景深邃渐变
    const grad = ctx.createRadialGradient(w/2, h/2, 0, w/2, h/2, w)
    grad.addColorStop(0, '#1a1525')
    grad.addColorStop(1, '#05020a')
    ctx.fillStyle = grad
    ctx.fillRect(0, 0, w, h)

    stars.forEach(s => {
      s.alpha += s.speed
      if (s.alpha > 1 || s.alpha < 0.1) s.speed *= -1

      // 视差位移计算
      const dx = mousePos.value.x * 30 * s.z
      const dy = mousePos.value.y * 30 * s.z

      ctx.beginPath()
      ctx.arc(s.x + dx, s.y + dy, s.r, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(255, 255, 255, ${s.alpha * 0.8})`
      ctx.fill()
    })
    animId = requestAnimationFrame(animate)
  }
  animate()

  onUnmounted(() => {
    cancelAnimationFrame(animId)
    window.removeEventListener('mousemove', handleMouseMove)
  })
})
</script>

<template>
  <canvas ref="canvas" class="star-bg"></canvas>
</template>

<style scoped>
.star-bg {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  z-index: 0;
  pointer-events: none;
}
</style>