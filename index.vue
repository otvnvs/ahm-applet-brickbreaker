<template>
  <div class="a">
    <header class="b">
      <h1>Breaker</h1>
      <p>Score: <b>{{ s }}</b> | Lives: <b>{{ v }}</b></p>
    </header>

    <main class="c" ref="m" @mousemove="mm" @touchmove="tm">
      <div v-if="o" class="o" @click="r">
        <h2>{{ v <= 0 ? 'GAME OVER' : 'YOU WIN!' }}</h2>
        <p>Tap to play again</p>
      </div>
      <canvas ref="g" width="320" height="400"></canvas>
    </main>

    <footer class="f">
      <p class="h">Drag screen to slide paddle</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const s = ref(0), v = ref(3), o = ref(false), g = ref(null), m = ref(null)
let c, ctx, t, p = 140, b = { x: 160, y: 300, dx: 2, dy: -3, r: 5 }, k = []

const r = () => {
  s.value = 0; v.value = 3; o.value = false; p = 140
  b = { x: 160, y: 300, dx: 2, dy: -4, r: 5 }; k = []
  for (let r = 0; r < 4; r++) {
    for (let l = 0; l < 6; l++) k.push({ x: l * 50 + 13, y: r * 20 + 40, w: 44, h: 14, a: 1 })
  }
}

const mm = (e) => {
  if (!c) return
  const r = c.getBoundingClientRect()
  p = Math.max(0, Math.min(260, (e.clientX - r.left) - 30))
}

const tm = (e) => {
  if (!c || !e.touches[0]) return
  const r = c.getBoundingClientRect()
  p = Math.max(0, Math.min(260, (e.touches[0].clientX - r.left) - 30))
}

const u = () => {
  if (o.value) return
  ctx.fillStyle = '#111112'
  ctx.fillRect(0, 0, 320, 400)

  // Ball Movement & Boundary Physics
  b.x += b.dx; b.y += b.dy
  if (b.x - b.r < 0 || b.x + b.r > 320) b.dx *= -1
  if (b.y - b.r < 0) b.dy *= -1

  // Paddle Collision Loop
  if (b.y + b.r >= 370 && b.y - b.r <= 378 && b.x >= p && b.x <= p + 60) {
    b.dy = -Math.abs(b.dy)
    b.dx = (b.x - (p + 30)) * 0.15
  }

  // Floor Drop/Life Loss Handler
  if (b.y + b.r > 400) {
    v.value--
    if (v.value <= 0) o.value = true
    else b = { x: p + 30, y: 350, dx: 2, dy: -4, r: 5 }
  }

  // Render & Check Bricks Grid
  let w = true
  k.forEach(blk => {
    if (!blk.a) return
    w = false
    if (b.x > blk.x && b.x < blk.x + blk.w && b.y + b.r > blk.y && b.y - b.r < blk.y + blk.h) {
      b.dy *= -1; blk.a = 0; s.value += 10
    }
    ctx.fillStyle = '#3a3a3c'
    ctx.fillRect(blk.x, blk.y, blk.w, blk.h)
    ctx.strokeStyle = '#000'
    ctx.strokeRect(blk.x, blk.y, blk.w, blk.h)
  })
  if (w) o.value = true

  // Render Paddle 
  ctx.fillStyle = '#3a3a3c'
  ctx.fillRect(p, 370, 60, 8)

  // Render Ball
  ctx.beginPath()
  ctx.arc(b.x, b.y, b.r, 0, Math.PI * 2)
  ctx.fillStyle = '#fff'
  ctx.fill()
}

onMounted(() => {
  c = g.value; ctx = c.getContext('2d')
  r(); t = setInterval(u, 1000 / 60)
})
onUnmounted(() => clearInterval(t))
</script>

<style scoped>
.a { width: 100vw; height: 100vh; background: #000; color: #fff; display: flex; flex-direction: column; align-items: center; justify-content: space-between; padding: 20px; font-family: sans-serif; box-sizing: border-box; user-select: none; overflow: hidden; }
.b { text-align: center; margin-bottom: 5px; width: 100%; }
h1 { margin: 0; font-size: 20px; text-transform: uppercase; color: #e2e8f0; letter-spacing: 1.5px; }
p { margin: 4px 0; color: #71717a; font-size: 13px; }
b { color: #fff; }
.c { width: 100%; max-width: 320px; aspect-ratio: 320 / 400; position: relative; border: 2px solid #1c1c1e; box-shadow: 0 20px 40px rgba(0,0,0,0.6); background: #111112; }
canvas { display: block; width: 100%; height: 100%; }
.o { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.95); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; cursor: pointer; }
.o h2 { color: #ef4444; margin: 0; font-size: 22px; letter-spacing: 1px; }
.o p { color: #71717a; font-size: 12px; margin-top: 6px; }
.f { width: 100%; text-align: center; }
.h { font-size: 11px; text-transform: uppercase; letter-spacing: 0.5px; color: #3a3a3c; }
@media (max-width: 360px) { .c { max-width: 90vw; } }
</style>
