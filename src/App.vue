<template>
  <div class="page" @mousemove="handleMouseMove" @mouseleave="handleMouseLeave">
    <canvas ref="waterCanvas" class="water-canvas"></canvas>
    <canvas ref="particleCanvas" class="particle-canvas"></canvas>
    <div class="overlay"></div>
    <div class="spotlight" :style="spotlightStyle"></div>

    <main class="content-shell">
      <section class="hero-card glass-reflection reveal" :style="heroTransformStyle">
        <span class="glass-sheen"></span>
        <div class="hero-top">
          <div>
            <p class="eyebrow">Professional Cover Letter</p>
            <h1>Nasir Hill</h1>
            <p class="subtitle">
              Computer Science Graduate • IT Technician • Cybersecurity-Focused Problem Solver
            </p>
          </div>

          <div class="contact-card reveal reveal-delay-1">
            <a href="mailto:NAH6695@gmail.com">NAH6695@gmail.com</a>
            <p>(706) 358-9099</p>
            <p>Huntsville, Alabama</p>
            <a href="https://github.com/Nasirthemad" target="_blank" rel="noreferrer">GitHub: Nasirthemad</a>
          </div>
        </div>

        <div class="divider reveal reveal-delay-1"></div>

        <article class="letter-grid">
          <div class="letter-main reveal reveal-delay-1">
              <p>Dear Hiring Manager,</p>

            <p>
              I am excited to bring my background in IT support, cybersecurity, and full-stack development to an organization that values secure, scalable, and user-focused technology. As a Computer Science graduate from Alabama A&amp;M University, I have developed a strong foundation in systems, networking, and modern application development, with a growing focus on building intelligent and secure software solutions.
            </p>

            <p>
              In my role as an <strong>IT Technician at Fiscal Systems Inc.</strong>, I worked extensively with Linux-based systems, remote troubleshooting tools such as VNC, and network diagnostics to resolve real-world technical issues. I analyzed transaction logs, maintained system updates, managed IP configurations, and supported mission-critical systems used in production environments. This experience strengthened my ability to troubleshoot under pressure, communicate effectively with users, and ensure system reliability and performance.
            </p>

            <p>
              My cybersecurity experience through the <strong>National Defense Education Program</strong> allowed me to work with Oracle VM, Ngrok, and MITRE ATT&amp;CK-aligned techniques to simulate phishing attacks and educate users on security awareness. This reinforced my understanding of both offensive and defensive security practices and deepened my interest in secure system design.
            </p>

            <p>
              Beyond my professional experience, I have built advanced full-stack applications, including my <strong>AI Security+ Study Platform</strong>. This project features authentication with Supabase, real-time AI-generated cybersecurity questions using the OpenAI API, a dynamic quiz engine with 90-question exam simulations, performance tracking dashboards, and cloud-synced user data. I designed the system with a scalable architecture separating frontend, backend, and database layers, and deployed it in a production environment with secure API handling.
            </p>

            <p>
              I am particularly interested in opportunities where I can combine my skills in IT support, cybersecurity, and software development to build efficient, secure, and intelligent systems. I bring a strong work ethic, adaptability, and a continuous learning mindset, and I am eager to contribute to a team where I can grow and make an impact.
            </p>

            <p>
              Thank you for your time and consideration. I look forward to the opportunity to discuss how my background and skills align with your team's goals.
            </p>

            <p class="signature">Sincerely,<br />Nasir Hill</p>
          </div>

          <aside class="highlights-panel">
            <div class="panel-card float-card delay-1 reveal reveal-delay-1 glass-reflection">
              <span class="glass-sheen"></span>
              <span class="panel-label">Core Strengths</span>
              <ul>
                <li>IT troubleshooting & incident response</li>
                <li>Cybersecurity awareness & phishing simulation</li>
                <li>Linux, VMs, networking, and remote support</li>
                <li>JavaScript, Python, C++, Node.js, and SQL</li>
              </ul>
            </div>

            <div class="panel-card float-card delay-2 reveal reveal-delay-2 glass-reflection">
              <span class="glass-sheen"></span>
              <span class="panel-label">Professional Experience</span>
              <p><strong>Fiscal Systems Inc.</strong><br />IT Technician<br />March 2025 – March 2026</p>
              <p><strong>NDEP</strong><br />Cybersecurity Intern<br />May 2024 – August 2024</p>
            </div>

            <div class="panel-card float-card delay-3 reveal reveal-delay-3 glass-reflection">
              <span class="glass-sheen"></span>
              <span class="panel-label">Featured Project</span>
              <p>
                <strong>AI Security+ Study Bot</strong><br />A full-stack study assistant with quiz flow, clickable answers, explanations, and saved progress.
              </p>
            </div>
          </aside>
        </article>
      </section>
    </main>
  </div>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const waterCanvas = ref(null)
const particleCanvas = ref(null)
const mouse = ref({ x: 0.5, y: 0.5, active: false })
const tilt = ref({ x: 0, y: 0 })

let waterAnimationFrameId = null
let particleAnimationFrameId = null
let ctx = null
let particleCtx = null
let width = 0
let height = 0
let dpr = 1
let ripples = []
let particles = []
let waveTime = 0
let revealObserver = null

const heroTransformStyle = computed(() => ({
  transform: `perspective(1400px) rotateX(${tilt.value.y}deg) rotateY(${tilt.value.x}deg) translateY(-4px)`
}))

const spotlightStyle = computed(() => ({
  left: `${mouse.value.x * 100}%`,
  top: `${mouse.value.y * 100}%`,
  opacity: mouse.value.active ? 1 : 0.65
}))

function resizeCanvases() {
  dpr = window.devicePixelRatio || 1
  width = window.innerWidth
  height = Math.max(window.innerHeight, document.documentElement.scrollHeight)

  ;[waterCanvas.value, particleCanvas.value].forEach((canvas) => {
    if (!canvas) return
    canvas.width = width * dpr
    canvas.height = height * dpr
    canvas.style.width = `${width}px`
    canvas.style.height = `${height}px`
  })

  ctx = waterCanvas.value?.getContext('2d') || null
  particleCtx = particleCanvas.value?.getContext('2d') || null

  if (ctx) ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
  if (particleCtx) particleCtx.setTransform(dpr, 0, 0, dpr, 0, 0)

  createParticles()
}

function addRipple(x, y, strength = 1.85) {
  ripples.push({
    x,
    y,
    radius: 8,
    alpha: 0.35 * strength,
    lineWidth: 2 + strength,
    speed: 2.5 + strength * 1.2
  })

  if (ripples.length > 34) ripples.shift()
}

function handleMouseMove(event) {
  const x = event.clientX / window.innerWidth
  const y = event.clientY / window.innerHeight

  mouse.value = { x, y, active: true }
  tilt.value = {
    x: (x - 0.5) * 9,
    y: (0.5 - y) * 9
  }

  if (Math.random() > 0.62) addRipple(event.clientX, event.clientY + window.scrollY, 1.4)
}

function handleMouseLeave() {
  mouse.value.active = false
  tilt.value = { x: 0, y: 0 }
}

function drawWaterGradient() {
  const gradient = ctx.createLinearGradient(0, 0, width, height)
  gradient.addColorStop(0, '#000000')
  gradient.addColorStop(0.5, '#050505')
  gradient.addColorStop(1, '#000000')
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, width, height)
}

function drawShaderMesh() {
  const cols = 20
  const rows = 12
  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < cols; col++) {
      const x = (col / (cols - 1)) * width
      const yBase = height * 0.58 + (row / (rows - 1)) * (height * 0.42)
      const distortion =
        Math.sin(col * 0.65 + waveTime * 1.4 + mouse.value.x * 3) * 10 +
        Math.cos(row * 0.85 + waveTime * 1.1 + mouse.value.y * 4) * 10
      const y = yBase + distortion

      const nextX = ((col + 1) / (cols - 1)) * width
      const nextYBase = height * 0.58 + (row / (rows - 1)) * (height * 0.42)
      const nextDistortion =
        Math.sin((col + 1) * 0.65 + waveTime * 1.4 + mouse.value.x * 3) * 10 +
        Math.cos(row * 0.85 + waveTime * 1.1 + mouse.value.y * 4) * 10
      const nextY = nextYBase + nextDistortion

      if (col < cols - 1) {
        ctx.beginPath()
        ctx.moveTo(x, y)
        ctx.lineTo(nextX, nextY)
        ctx.strokeStyle = 'rgba(135, 220, 255, 0.08)'
        ctx.lineWidth = 1
        ctx.stroke()
      }

      if (row < rows - 1) {
        const downYBase = height * 0.58 + ((row + 1) / (rows - 1)) * (height * 0.42)
        const downDistortion =
          Math.sin(col * 0.65 + waveTime * 1.4 + mouse.value.x * 3) * 10 +
          Math.cos((row + 1) * 0.85 + waveTime * 1.1 + mouse.value.y * 4) * 10
        const downY = downYBase + downDistortion
        ctx.beginPath()
        ctx.moveTo(x, y)
        ctx.lineTo(x, downY)
        ctx.strokeStyle = 'rgba(135, 220, 255, 0.05)'
        ctx.lineWidth = 1
        ctx.stroke()
      }
    }
  }
}

function drawWaveLayer(amplitude, wavelength, speed, alpha, offsetY) {
  ctx.beginPath()
  ctx.moveTo(0, height)

  const mouseInfluence = (mouse.value.x - 0.5) * 120

  for (let x = 0; x <= width; x += 8) {
    const y =
      offsetY +
      Math.sin(x / wavelength + waveTime * speed + mouseInfluence / 120) * amplitude +
      Math.cos(x / (wavelength * 0.75) + waveTime * speed * 0.8) * (amplitude * 0.45)

    ctx.lineTo(x, y)
  }

  ctx.lineTo(width, height)
  ctx.closePath()
  ctx.fillStyle = `rgba(160, 220, 255, ${alpha})`
  ctx.fill()
}

function drawHighlights() {
  const glowX = mouse.value.active ? mouse.value.x * width : width * 0.7
  const glowY = mouse.value.active ? mouse.value.y * window.innerHeight + window.scrollY : height * 0.2
  const radial = ctx.createRadialGradient(glowX, glowY, 0, glowX, glowY, 220)
  radial.addColorStop(0, 'rgba(200, 240, 255, 0.45)')
  radial.addColorStop(0.4, 'rgba(200, 240, 255, 0.18)')
  radial.addColorStop(1, 'rgba(200, 240, 255, 0)')
  ctx.fillStyle = radial
  ctx.fillRect(0, 0, width, height)
}

function drawRipples() {
  ripples = ripples.filter((ripple) => ripple.alpha > 0.003)

  for (const ripple of ripples) {
    ctx.beginPath()
    ctx.arc(ripple.x, ripple.y, ripple.radius, 0, Math.PI * 2)
    ctx.strokeStyle = `rgba(220, 250, 255, ${ripple.alpha})`
    ctx.lineWidth = ripple.lineWidth
    ctx.stroke()

    ctx.beginPath()
    ctx.arc(ripple.x, ripple.y, ripple.radius * 0.55, 0, Math.PI * 2)
    ctx.strokeStyle = `rgba(160, 235, 255, ${ripple.alpha * 0.45})`
    ctx.lineWidth = Math.max(1, ripple.lineWidth * 0.5)
    ctx.stroke()

    ripple.radius += ripple.speed
    ripple.alpha *= 0.972
  }
}

function animateWater() {
  if (!ctx) return
  waveTime += 0.016

  drawWaterGradient()
  drawHighlights()
  drawShaderMesh()
  drawWaveLayer(18, 150, 1.35, 0.1, height * 0.72)
  drawWaveLayer(24, 210, 1.0, 0.12, height * 0.79)
  drawWaveLayer(32, 280, 0.66, 0.15, height * 0.86)
  drawRipples()

  waterAnimationFrameId = window.requestAnimationFrame(animateWater)
}

function createParticles() {
  const total = Math.min(90, Math.floor((window.innerWidth * window.innerHeight) / 24000))
  particles = Array.from({ length: total }, () => ({
    x: Math.random() * width,
    y: Math.random() * height,
    size: Math.random() * 1.8 + 0.5,
    speedX: (Math.random() - 0.5) * 0.28,
    speedY: Math.random() * 0.35 + 0.08,
    alpha: Math.random() * 0.35 + 0.08
  }))
}

function animateParticles() {
  if (!particleCtx) return

  particleCtx.clearRect(0, 0, width, height)

  for (const p of particles) {
    const dx = mouse.value.x * width - p.x
    const dy = (mouse.value.y * window.innerHeight + window.scrollY) - p.y
    const dist = Math.hypot(dx, dy)
    const influence = dist < 180 ? (180 - dist) / 180 : 0

    p.x += p.speedX - dx * 0.00012 * influence
    p.y += p.speedY - dy * 0.00012 * influence

    if (p.y > height + 10) {
      p.y = -10
      p.x = Math.random() * width
    }
    if (p.x < -10) p.x = width + 10
    if (p.x > width + 10) p.x = -10

    particleCtx.beginPath()
    particleCtx.arc(p.x, p.y, p.size, 0, Math.PI * 2)
    particleCtx.fillStyle = `rgba(210, 240, 255, ${p.alpha})`
    particleCtx.fill()
  }

  particleAnimationFrameId = window.requestAnimationFrame(animateParticles)
}

function initScrollReveal() {
  const items = document.querySelectorAll('.reveal')
  revealObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) entry.target.classList.add('revealed')
      })
    },
    { threshold: 0.14 }
  )
  items.forEach((item) => revealObserver.observe(item))
}

onMounted(() => {
  resizeCanvases()
  createParticles()
  animateWater()
  animateParticles()
  initScrollReveal()
  window.addEventListener('resize', resizeCanvases)

  const seedRipples = [
    [window.innerWidth * 0.22, window.innerHeight * 0.72, 1.1],
    [window.innerWidth * 0.78, window.innerHeight * 0.67, 1.25],
    [window.innerWidth * 0.54, window.innerHeight * 0.8, 1.2]
  ]
  seedRipples.forEach(([x, y, strength]) => addRipple(x, y, strength))
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', resizeCanvases)
  if (waterAnimationFrameId) window.cancelAnimationFrame(waterAnimationFrameId)
  if (particleAnimationFrameId) window.cancelAnimationFrame(particleAnimationFrameId)
  if (revealObserver) revealObserver.disconnect()
})
</script>

<style scoped>
:global(*) {
  box-sizing: border-box;
}

:global(html, body, #app) {
  min-height: 100%;
}

:global(body) {
  margin: 0;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  background: #000;
  color: #eaf6ff;
}

.page {
  position: relative;
  min-height: 100vh;
  overflow: hidden;
  background:
    radial-gradient(circle at top, rgba(255,255,255,0.06), transparent 30%),
    linear-gradient(180deg, #000000 0%, #020202 100%);
}

.water-canvas,
.particle-canvas,
.overlay,
.spotlight {
  position: absolute;
  inset: 0;
}

.water-canvas {
  z-index: 0;
}

.particle-canvas {
  z-index: 1;
  pointer-events: none;
}

.overlay {
  z-index: 2;
  pointer-events: none;
  background:
    linear-gradient(to bottom, rgba(0, 0, 0, 0.16), rgba(0, 0, 0, 0.42)),
    radial-gradient(circle at 20% 20%, rgba(255, 255, 255, 0.04), transparent 20%);
  backdrop-filter: blur(1px);
}

.spotlight {
  z-index: 3;
  pointer-events: none;
  width: 36rem;
  height: 36rem;
  transform: translate(-50%, -50%);
  background: radial-gradient(circle, rgba(190,235,255,0.16) 0%, rgba(190,235,255,0.08) 22%, rgba(190,235,255,0.03) 42%, rgba(190,235,255,0) 72%);
  filter: blur(10px);
  transition: left 120ms linear, top 120ms linear, opacity 220ms ease;
}

.content-shell {
  position: relative;
  z-index: 4;
  width: min(1180px, calc(100% - 32px));
  margin: 0 auto;
  padding: 42px 0 56px;
}

.hero-card {
  position: relative;
  overflow: hidden;
  padding: 34px;
  border: 1px solid rgba(209, 236, 255, 0.18);
  border-radius: 28px;
  background: linear-gradient(180deg, rgba(8, 22, 38, 0.74), rgba(5, 13, 23, 0.7));
  backdrop-filter: blur(20px) saturate(130%);
  box-shadow:
    0 24px 80px rgba(0, 0, 0, 0.45),
    inset 0 1px 0 rgba(255, 255, 255, 0.08),
    inset 0 -1px 0 rgba(255, 255, 255, 0.04);
  transition: transform 180ms ease-out, box-shadow 240ms ease;
}

.glass-reflection::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, rgba(255,255,255,0.12), rgba(255,255,255,0.02) 28%, transparent 46%);
  pointer-events: none;
}

.glass-sheen {
  position: absolute;
  inset: -20%;
  background: linear-gradient(120deg, transparent 20%, rgba(255,255,255,0.12) 38%, rgba(255,255,255,0.03) 48%, transparent 58%);
  transform: translateX(-35%) rotate(8deg);
  animation: sheenMove 9s linear infinite;
  pointer-events: none;
}

.hero-top {
  display: grid;
  grid-template-columns: 1.6fr 0.9fr;
  gap: 24px;
  align-items: start;
}

.eyebrow {
  margin: 0 0 8px;
  font-size: 0.84rem;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: #91d7ff;
}

h1 {
  margin: 0;
  font-size: clamp(2.3rem, 4vw, 4.2rem);
  line-height: 0.95;
  letter-spacing: -0.04em;
}

.subtitle {
  margin: 14px 0 0;
  max-width: 700px;
  color: rgba(230, 244, 255, 0.82);
  font-size: 1.02rem;
  line-height: 1.65;
}

.contact-card,
.panel-card {
  position: relative;
  overflow: hidden;
  border: 1px solid rgba(181, 227, 255, 0.14);
  background: rgba(255, 255, 255, 0.055);
  box-shadow:
    0 20px 35px rgba(0, 0, 0, 0.24),
    inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

.contact-card {
  justify-self: end;
  width: min(100%, 300px);
  padding: 18px 20px;
  border-radius: 22px;
}

.contact-card p,
.contact-card a {
  display: block;
  margin: 0 0 10px;
  color: #eaf6ff;
  text-decoration: none;
  line-height: 1.55;
  word-break: break-word;
}

.contact-card a:hover {
  color: #9ddfff;
}

.divider {
  height: 1px;
  margin: 28px 0;
  background: linear-gradient(90deg, rgba(145, 215, 255, 0.55), rgba(145, 215, 255, 0.08));
}

.letter-grid {
  display: grid;
  grid-template-columns: minmax(0, 1.7fr) minmax(280px, 0.9fr);
  gap: 24px;
}

.letter-main {
  padding: 6px 4px 6px 2px;
}

.letter-main p {
  margin: 0 0 20px;
  color: rgba(237, 247, 255, 0.92);
  line-height: 1.86;
  font-size: 1rem;
}

.signature {
  margin-top: 30px;
  color: #c5ebff;
}

.highlights-panel {
  display: grid;
  gap: 18px;
  align-content: start;
}

.panel-card {
  padding: 20px;
  border-radius: 24px;
}

.panel-card p,
.panel-card li {
  color: rgba(232, 246, 255, 0.86);
  line-height: 1.7;
}

.panel-card ul {
  margin: 12px 0 0;
  padding-left: 18px;
}

.panel-label {
  display: inline-block;
  margin-bottom: 10px;
  color: #91d7ff;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.16em;
  text-transform: uppercase;
}

.float-card {
  animation: floatCard 7s ease-in-out infinite;
}

.delay-1 { animation-delay: 0s; }
.delay-2 { animation-delay: 1.2s; }
.delay-3 { animation-delay: 2.2s; }

.reveal {
  opacity: 0;
  transform: translateY(32px) scale(0.985);
  transition: opacity 800ms ease, transform 900ms cubic-bezier(0.22, 1, 0.36, 1);
}

.revealed {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.reveal-delay-1 { transition-delay: 120ms; }
.reveal-delay-2 { transition-delay: 240ms; }
.reveal-delay-3 { transition-delay: 360ms; }

@keyframes floatCard {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-8px); }
}

@keyframes sheenMove {
  0% { transform: translateX(-38%) rotate(8deg); opacity: 0.45; }
  50% { opacity: 0.75; }
  100% { transform: translateX(42%) rotate(8deg); opacity: 0.45; }
}

@media (max-width: 920px) {
  .hero-top,
  .letter-grid {
    grid-template-columns: 1fr;
  }

  .contact-card {
    justify-self: start;
  }
}

@media (max-width: 640px) {
  .content-shell {
    width: min(100% - 18px, 1180px);
    padding-top: 18px;
    padding-bottom: 26px;
  }

  .hero-card {
    padding: 22px 18px;
    border-radius: 22px;
  }

  .spotlight {
    width: 24rem;
    height: 24rem;
  }

  h1 {
    font-size: 2.25rem;
  }

  .subtitle,
  .letter-main p,
  .panel-card p,
  .panel-card li,
  .contact-card p,
  .contact-card a {
    font-size: 0.95rem;
  }
}
</style>
