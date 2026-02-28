<template>
  <div class="stage" @mousemove="onMove" @mouseleave="onLeave">
    <ParticleBackground :pointer="pointer" />

    <!-- Ambient blobs -->
    <div class="blobs" aria-hidden="true">
      <div class="blob b1"></div>
      <div class="blob b2"></div>
      <div class="blob b3"></div>
    </div>

    <!-- Cursor spotlight -->
    <div class="spotlight" aria-hidden="true"></div>

    <!-- Subtle grain overlay -->
    <div class="grain" aria-hidden="true"></div>

    <main class="content" :class="{ ready }">
      <section class="hero" role="region" aria-label="BxPLAB">
        <div class="badge">IT-партнёр для бизнеса</div>

        <div class="logoOrb" :style="{ '--tiltX': tilt.x + 'deg', '--tiltY': tilt.y + 'deg' }">
          <div class="ring" aria-hidden="true"></div>
          <div class="ring ring2" aria-hidden="true"></div>

          <div class="logoShell">
            <img class="logo" :src="logoUrl" alt="BxPLAB logo" />
          </div>
        </div>

        <p class="lead">Мы берём IT на себя, чтобы вы могли заниматься бизнесом.</p>

        <div class="cta">
          <a class="ctaBtn" :href="tgLink" target="_blank" rel="noreferrer">
            <span class="ctaIcon" aria-hidden="true">
              <svg viewBox="0 0 24 24" width="18" height="18" fill="none">
                <path d="M21.5 4.6c.5-.2.9.2.7.8l-3.3 15.3c-.2 1.1-1 1.4-1.9.9l-4.9-3.6-2.4 2.3c-.3.3-.6.5-1 .5l.4-5.8L17 7.2c.4-.4-.1-.6-.6-.3l-9.7 6.1-4.2-1.3c-1.1-.3-1.1-1.1.2-1.6L21.5 4.6Z"
                      fill="currentColor" opacity="0.92"/>
              </svg>
            </span>
            Написать в Telegram
          </a>

          <div class="hint">
            <span class="dot" aria-hidden="true"></span>
            <span>t.me/bxp_lab</span>
          </div>
        </div>

        <p class="soon">Скоро здесь будет полноценный сайт</p>
      </section>
    </main>
  </div>
</template>

<script setup>
import { computed, onMounted, reactive, ref } from 'vue'
import ParticleBackground from './ParticleBackground.vue'
import logoUrl from '../assets/logo.png'

const tgLink = 'https://t.me/bxp_lab'

const ready = ref(false)

// pointer for particle interaction + spotlight
const pointer = reactive({ x: 0, y: 0, inside: false })

const tilt = reactive({ x: 0, y: 0 })

const reduceMotion = computed(() =>
  typeof window !== 'undefined'
    ? window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches
    : false
)

function setSpotlight(x, y){
  document.documentElement.style.setProperty('--mx', x + 'px')
  document.documentElement.style.setProperty('--my', y + 'px')
}

function onMove(e){
  const w = window.innerWidth || 1
  const h = window.innerHeight || 1
  const nx = (e.clientX / w) * 2 - 1
  const ny = (e.clientY / h) * 2 - 1

  pointer.x = e.clientX
  pointer.y = e.clientY
  pointer.inside = true

  setSpotlight(e.clientX, e.clientY)

  if (!reduceMotion.value){
    // gentle tilt (clamped)
    tilt.y = Math.max(-10, Math.min(10, nx * 9))
    tilt.x = Math.max(-10, Math.min(10, -ny * 9))
  }
}

function onLeave(){
  pointer.inside = false
  if (!reduceMotion.value){
    tilt.x = 0
    tilt.y = 0
  }
}

onMounted(() => {
  // initial spotlight to center
  setSpotlight(window.innerWidth * 0.5, window.innerHeight * 0.35)

  // staged reveal
  requestAnimationFrame(() => {
    setTimeout(() => { ready.value = true }, 140)
  })
})
</script>

<style scoped>
.stage{
  position: relative;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}

.content{
  position: relative;
  height: 100%;
  display: grid;
  place-items: center;
  padding: 24px;
  z-index: 5;
}

/* Main hero block (no big card, more airy) */
.hero{
  width: min(720px, 100%);
  text-align: center;
  padding: clamp(20px, 3.6vw, 34px);
  border-radius: 36px;

  background: rgba(255,255,255,0.52);
  border: 1px solid rgba(0,0,0,0.10);
  box-shadow:
    0 40px 120px rgba(0,0,0,0.10),
    0 10px 30px rgba(0,0,0,0.06);
  backdrop-filter: blur(18px);

  opacity: 0;
  transform: translateY(12px) scale(0.99);
}

/* Badge */
.badge{
  display: inline-flex;
  gap: 10px;
  align-items: center;
  padding: 10px 14px;
  border-radius: 999px;
  border: 1px solid rgba(0,0,0,0.10);
  background: rgba(255,255,255,0.70);
  box-shadow: 0 12px 34px rgba(0,0,0,0.06);
  color: rgba(0,0,0,0.70);
  font-size: 14px;
  letter-spacing: 0.01em;
}

.logoOrb{
  margin: 22px auto 0;
  width: clamp(220px, 30vw, 320px);
  height: clamp(220px, 30vw, 320px);
  position: relative;
  transform-style: preserve-3d;
  transform: perspective(1100px) rotateX(var(--tiltX)) rotateY(var(--tiltY));
  transition: transform 420ms cubic-bezier(.2,.8,.2,1);
}

.logoShell{
  position: absolute;
  inset: 18px;
  border-radius: 34px;
  background: rgba(255,255,255,0.62);
  border: 1px solid rgba(0,0,0,0.10);
  box-shadow:
    0 30px 90px rgba(0,0,0,0.10),
    inset 0 1px 0 rgba(255,255,255,0.9);
  display: grid;
  place-items: center;
  overflow: hidden;
}

.logo{
  width: 78%;
  height: 78%;
  object-fit: contain;
  filter: drop-shadow(0 24px 34px rgba(0,0,0,0.18));
  transform: translateZ(26px);
}

/* Animated rings */
.ring{
  position: absolute;
  inset: -8px;
  border-radius: 44px;
  background: conic-gradient(
    from 180deg,
    rgba(255, 170, 120, 0.0),
    rgba(255, 170, 120, 0.55),
    rgba(140, 190, 255, 0.55),
    rgba(255, 170, 120, 0.0)
  );
  filter: blur(10px);
  opacity: 0.85;
  animation: spin 8s linear infinite;
}

.ring2{
  inset: -20px;
  opacity: 0.55;
  filter: blur(18px);
  animation-duration: 13s;
  animation-direction: reverse;
}

@keyframes spin{
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.lead{
  margin: 18px auto 0;
  max-width: 52ch;
  font-size: clamp(16px, 2.1vw, 18px);
  line-height: 1.5;
  color: rgba(0,0,0,0.72);
  opacity: 0;
  transform: translateY(8px);
}

.cta{
  margin-top: 22px;
  display: grid;
  justify-items: center;
  gap: 12px;
  opacity: 0;
  transform: translateY(8px);
}

.ctaBtn{
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 14px 18px;
  border-radius: 999px;
  border: 1px solid rgba(0,0,0,0.14);
  background: rgba(255,255,255,0.74);
  box-shadow: 0 18px 50px rgba(0,0,0,0.10);
  transition: transform 180ms ease, box-shadow 180ms ease, background 180ms ease;
  user-select: none;
  color: rgba(0,0,0,0.86);
}

.ctaBtn:hover{
  transform: translateY(-2px);
  box-shadow: 0 26px 70px rgba(0,0,0,0.14);
  background: rgba(255,255,255,0.86);
}

.ctaIcon{
  display: inline-flex;
  width: 34px;
  height: 34px;
  border-radius: 999px;
  align-items: center;
  justify-content: center;
  background: rgba(0,0,0,0.06);
  border: 1px solid rgba(0,0,0,0.10);
}

.hint{
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 8px 12px;
  border-radius: 999px;
  background: rgba(255,255,255,0.55);
  border: 1px solid rgba(0,0,0,0.10);
  color: rgba(0,0,0,0.58);
  font-size: 13px;
}

.dot{
  width: 8px;
  height: 8px;
  border-radius: 999px;
  background: rgba(0,0,0,0.35);
  box-shadow: 0 0 0 6px rgba(0,0,0,0.06);
}

.soon{
  margin: 16px 0 0;
  font-size: 13px;
  color: rgba(0,0,0,0.52);
  opacity: 0;
  transform: translateY(8px);
}

/* Ambient blobs */
.blobs{
  position:absolute;
  inset:0;
  z-index:1;
  pointer-events:none;
}
.blob{
  position:absolute;
  width: 680px;
  height: 680px;
  border-radius: 999px;
  filter: blur(52px);
  opacity: 0.50;
  transform: translate3d(0,0,0);
  animation: float 12s ease-in-out infinite;
  background: radial-gradient(circle at 30% 30%, rgba(255, 170, 120, 0.82), rgba(255,255,255,0) 60%);
}
.b1{ left: -260px; top: -260px; }
.b2{
  right: -300px; bottom: -320px;
  width: 760px; height: 760px;
  opacity: 0.42;
  animation-duration: 14s;
  background: radial-gradient(circle at 35% 35%, rgba(140, 190, 255, 0.82), rgba(255,255,255,0) 62%);
}
.b3{
  left: 35%;
  top: 60%;
  width: 520px; height: 520px;
  opacity: 0.30;
  animation-duration: 16s;
  background: radial-gradient(circle at 35% 35%, rgba(210, 150, 255, 0.76), rgba(255,255,255,0) 62%);
}
@keyframes float{
  0%,100%{ transform: translate(0px, 0px) scale(1); }
  50%{ transform: translate(28px, 34px) scale(1.04); }
}

/* Spotlight follows cursor */
.spotlight{
  position:absolute;
  inset:0;
  z-index:2;
  pointer-events:none;
  background:
    radial-gradient(420px 320px at var(--mx, 50%) var(--my, 35%),
      rgba(255, 255, 255, 0.0) 0%,
      rgba(255, 255, 255, 0.0) 35%,
      rgba(0, 0, 0, 0.05) 100%);
  mix-blend-mode: multiply;
}

/* Grain */
.grain{
  position:absolute;
  inset:-40%;
  z-index:4;
  pointer-events:none;
  opacity: 0.06;
  background-image:
    url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.9' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='160' height='160' filter='url(%23n)' opacity='.55'/%3E%3C/svg%3E");
  transform: rotate(2deg);
}

/* Reveal sequence */
.content.ready .hero{
  opacity: 1;
  transform: translateY(0) scale(1);
  transition: opacity 560ms ease, transform 560ms cubic-bezier(.2,.8,.2,1);
}
.content.ready .lead{
  opacity: 1;
  transform: translateY(0);
  transition: opacity 520ms ease 180ms, transform 520ms cubic-bezier(.2,.8,.2,1) 180ms;
}
.content.ready .cta{
  opacity: 1;
  transform: translateY(0);
  transition: opacity 520ms ease 300ms, transform 520ms cubic-bezier(.2,.8,.2,1) 300ms;
}
.content.ready .soon{
  opacity: 1;
  transform: translateY(0);
  transition: opacity 520ms ease 420ms, transform 520ms cubic-bezier(.2,.8,.2,1) 420ms;
}

@media (max-width: 520px){
  .hero{ border-radius: 30px; }
  .logoShell{ border-radius: 28px; }
  .ring{ border-radius: 40px; }
  .ring2{ border-radius: 46px; }
}

@media (prefers-reduced-motion: reduce){
  .blob{ animation: none; }
  .ring{ animation: none; }
  .logoOrb{ transition: none; }
  .spotlight{ display:none; }
}
</style>
