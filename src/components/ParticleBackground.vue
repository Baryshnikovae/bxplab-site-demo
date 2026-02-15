<template>
  <canvas ref="c" class="canvas" aria-hidden="true"></canvas>
</template>

<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const props = defineProps({
  pointer: { type: Object, required: true } // {x,y,inside}
})

const c = ref(null)

let ctx = null
let raf = 0
let w = 0
let h = 0
let dpr = 1
let particles = []
let lastT = performance.now()

const reduced = () =>
  window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches

function resize(){
  if (!c.value) return
  dpr = Math.max(1, Math.min(2, window.devicePixelRatio || 1))
  w = Math.floor(window.innerWidth)
  h = Math.floor(window.innerHeight)
  c.value.width = Math.floor(w * dpr)
  c.value.height = Math.floor(h * dpr)
  c.value.style.width = w + 'px'
  c.value.style.height = h + 'px'
  ctx = c.value.getContext('2d', { alpha: true })
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)

  // particle count proportional to area (capped)
  const target = Math.max(28, Math.min(86, Math.floor((w*h)/32000)))
  if (particles.length === 0){
    particles = makeParticles(target)
  } else if (particles.length < target){
    particles.push(...makeParticles(target - particles.length))
  } else if (particles.length > target){
    particles = particles.slice(0, target)
  }
}

function rand(a,b){ return a + Math.random()*(b-a) }

function makeParticles(n){
  const arr = []
  for (let i=0;i<n;i++){
    const warm = Math.random() < 0.60
    arr.push({
      x: rand(0, w),
      y: rand(0, h),
      vx: rand(-0.12, 0.12),
      vy: rand(-0.12, 0.12),
      r: rand(1.2, 3.2),
      baseA: rand(0.10, 0.26),
      tw: rand(0.002, 0.006), // twinkle speed
      ph: rand(0, Math.PI*2),
      hue: warm ? rand(18, 44) : rand(200, 226),
      flash: 0
    })
  }
  return arr
}

function draw(now){
  raf = requestAnimationFrame(draw)
  const dt = Math.min(32, now - lastT)
  lastT = now

  if (!ctx) return
  ctx.clearRect(0,0,w,h)

  // soft vignette
  const g = ctx.createRadialGradient(w*0.55, h*0.40, 80, w*0.55, h*0.40, Math.max(w,h)*0.85)
  g.addColorStop(0, 'rgba(255,255,255,0)')
  g.addColorStop(1, 'rgba(0,0,0,0.035)')
  ctx.fillStyle = g
  ctx.fillRect(0,0,w,h)

  const px = props.pointer?.x ?? -9999
  const py = props.pointer?.y ?? -9999
  const inside = !!props.pointer?.inside

  // update + draw particles
  for (const p of particles){
    if (!reduced()){
      p.x += p.vx * dt
      p.y += p.vy * dt

      // wrap
      if (p.x < -30) p.x = w + 30
      if (p.x > w + 30) p.x = -30
      if (p.y < -30) p.y = h + 30
      if (p.y > h + 30) p.y = -30

      // pointer interaction (gentle repel + micro flash)
      if (inside){
        const dx = p.x - px
        const dy = p.y - py
        const d2 = dx*dx + dy*dy
        const r = 160
        if (d2 < r*r){
          const d = Math.max(10, Math.sqrt(d2))
          const f = (1 - d/r) * 0.26
          p.x += (dx / d) * f * dt * 0.55
          p.y += (dy / d) * f * dt * 0.55
          p.flash = Math.min(1, p.flash + 0.02)
        }
      }

      // rare random flash
      if (Math.random() < 0.0016){
        p.flash = Math.min(1, p.flash + 0.75)
      }
      p.flash *= 0.985
    }

    const a = p.baseA + Math.sin(now * p.tw + p.ph) * 0.07 + p.flash * 0.25
    const radius = p.r + p.flash * 1.15

    // core
    ctx.beginPath()
    ctx.arc(p.x, p.y, radius, 0, Math.PI*2)
    ctx.fillStyle = `hsla(${p.hue}, 88%, 58%, ${Math.max(0, a)})`
    ctx.fill()

    // glow
    const gg = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, radius*6.5)
    gg.addColorStop(0, `hsla(${p.hue}, 92%, 62%, ${a*0.35})`)
    gg.addColorStop(1, `hsla(${p.hue}, 92%, 62%, 0)`)
    ctx.fillStyle = gg
    ctx.beginPath()
    ctx.arc(p.x, p.y, radius*6.5, 0, Math.PI*2)
    ctx.fill()
  }

  // subtle cursor trail + connecting lines to nearest particles
  if (!reduced() && inside){
    // soft disc
    ctx.fillStyle = 'rgba(0,0,0,0.018)'
    ctx.beginPath()
    ctx.arc(px, py, 46, 0, Math.PI*2)
    ctx.fill()

    // lines to nearest
    const maxD = 110
    let count = 0
    for (const p of particles){
      const dx = p.x - px
      const dy = p.y - py
      const d = Math.sqrt(dx*dx + dy*dy)
      if (d < maxD){
        const alpha = (1 - d/maxD) * 0.10
        ctx.strokeStyle = `rgba(0,0,0,${alpha})`
        ctx.lineWidth = 1
        ctx.beginPath()
        ctx.moveTo(px, py)
        ctx.lineTo(p.x, p.y)
        ctx.stroke()
        if (++count > 10) break
      }
    }
  }
}

onMounted(() => {
  resize()
  window.addEventListener('resize', resize, { passive: true })
  raf = requestAnimationFrame(draw)
})

onBeforeUnmount(() => {
  cancelAnimationFrame(raf)
  window.removeEventListener('resize', resize)
})
</script>

<style scoped>
.canvas{
  position:absolute;
  inset:0;
  width:100%;
  height:100%;
  z-index:0;
}
</style>
