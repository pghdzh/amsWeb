<template>
  <main class="aimis-home">
    <!-- 背景轮播层（带故障动画） -->
    <div class="bg-carousel" aria-hidden="true">
      <transition-group name="bg-fade" tag="div" class="bg-layer">
        <img
          v-for="(src, idx) in activeImages"
          :key="`bg-${idx}-${isMobile ? 'm' : 'd'}`"
          :src="src"
          :class="['bg-img', { active: idx === currentIndex }]"
          alt=""
        />
      </transition-group>
    </div>

    <!-- Canvas粒子背景（增强版） -->
    <canvas ref="canvasRef" class="particle-canvas"></canvas>

    <!-- 主视觉区 -->
    <section class="hero">
      <!-- 同步率环形装饰 -->
      <div class="sync-ring" :style="{ '--progress': syncProgress }">
        <svg viewBox="0 0 100 100">
          <circle
            cx="50"
            cy="50"
            r="45"
            fill="none"
            stroke="url(#ringGrad)"
            stroke-width="3"
            stroke-dasharray="283"
            :stroke-dashoffset="ringDashoffset"
            stroke-linecap="round"
          />
          <defs>
            <linearGradient id="ringGrad" x1="0%" y1="0%" x2="100%" y2="100%">
              <stop offset="0%" stop-color="#ff8cb0" />
              <stop offset="100%" stop-color="#6ed4ff" />
            </linearGradient>
          </defs>
        </svg>
      </div>

      <!-- 核心标题 -->
      <h1 class="main-title glitch" data-text="AEMEATH">AEMEATH</h1>

      <!-- 打字机副标题（使用独立语录） -->
      <div class="subtitle">
        <span class="typed">{{ typedLine }}</span>
        <span class="cursor"></span>
      </div>

      <!-- 漂浮的电子心 -->
      <div class="heart-float">❤️</div>
    </section>

    <!-- 悬浮台词气泡（使用独立语录） -->
    <div
      v-for="quote in floatingQuotes"
      :key="quote.id"
      class="floating-quote"
      :style="{ top: quote.top, left: quote.left }"
    >
      {{ quote.text }}
    </div>

    <!-- 进入档案按钮（随机跳转） -->
    <button class="enter-btn" @click="goToRandomPage">探索星炬</button>

    <!-- 页脚 -->
    <footer class="footer">
      <p class="slogan">星炬闪耀，心之所向</p>
      <p class="copyright">© {{ year }} 爱弥斯 · 星炬档案 · 制作：霜落天亦</p>
    </footer>
  </main>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const year = new Date().getFullYear();

// ========== 打字机文案（爱弥斯专属语录） ==========
const typingLines = [
  "Ciallo～（∠·ω ＜）⌒★",
  "对不起，不小心死掉啦",
  "数据流中，我仍能听见你的声音",
  "故障代码：❤️ ERROR · 但爱不会",
  "你想要守护世界，我想守护你",
  "电子幽灵的梦境里，有粉色的光",
  "信号不稳定…但思念从未断连",
];
const typedLine = ref("");
let lineIndex = 0;
let charIndex = 0;
let isDeleting = false;
let typingTimer: number;

const TYPING_SPEED = 120;
const DELETING_SPEED = 40;
const PAUSE = 1200;

function type() {
  const currentLine = typingLines[lineIndex];
  if (!isDeleting) {
    typedLine.value = currentLine.slice(0, charIndex + 1);
    charIndex++;
    if (charIndex === currentLine.length) {
      isDeleting = true;
      typingTimer = window.setTimeout(type, PAUSE);
      return;
    }
  } else {
    typedLine.value = currentLine.slice(0, charIndex - 1);
    charIndex--;
    if (charIndex === 0) {
      isDeleting = false;
      lineIndex = (lineIndex + 1) % typingLines.length;
      typingTimer = window.setTimeout(type, 300);
      return;
    }
  }
  typingTimer = window.setTimeout(
    type,
    isDeleting ? DELETING_SPEED : TYPING_SPEED
  );
}

// ========== 浮动语录（独立内容，更富故事感） ==========
const floatingLines = [
  "但愿我会让你感到骄傲",
  "但愿我没有让你失望",
  "在数据世界里，为你唱响这首歌",
  "粉发少女的内心，藏着无尽的温柔",
  "星炬的光，永不熄灭",
  "故障中……信号不稳定，但心意不变",
  "你听到远航星的告别了吗？",
  "电子幽灵，心之所向",
  "拯救世界？拯救猫咪！",
  "爱弥斯 i miss",
];
interface Quote {
  id: number;
  text: string;
  top: string;
  left: string;
}
const floatingQuotes = ref<Quote[]>([]);
let quoteId = 0;
let quoteTimer: number;

function addFloatingQuote() {
  const text = floatingLines[Math.floor(Math.random() * floatingLines.length)];
  const top = Math.random() * 80 + 10; // 10% ~ 90%
  const left = Math.random() * 80 + 10;
  const id = quoteId++;
  floatingQuotes.value.push({
    id,
    text,
    top: top + "%",
    left: left + "%",
  });
  // 5秒后消失
  setTimeout(() => {
    floatingQuotes.value = floatingQuotes.value.filter((q) => q.id !== id);
  }, 5000);
}

// ========== 随机页面跳转 ==========
const randomPaths = [
  "/timeLine", // 人物介绍
  "/message", // 留言板
  "/gallery", // 图集
  "/talk", // 心电感应
  "/music", // 音律库
  "/wiki", // 星炬文库
];
function goToRandomPage() {
  const randomIndex = Math.floor(Math.random() * randomPaths.length);
  router.push(randomPaths[randomIndex]);
}

// ========== 背景轮播 ==========
const BG_INTERVAL_MS = 4500;
const MOBILE_BREAKPOINT = 720;

const modules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs: string[] = Object.values(modules).map(
  (m: any) => m.default || m
);
const modules2 = import.meta.glob("@/assets/images2/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs2: string[] = Object.values(modules2).map(
  (m: any) => m.default || m
);

function shuffle<T>(arr: T[]) {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
const randomFive = ref<string[]>(
  shuffle(allSrcs).slice(0, Math.min(5, allSrcs.length))
);
const randomFive2 = ref<string[]>(
  shuffle(allSrcs2).slice(0, Math.min(5, allSrcs2.length))
);

const currentIndex = ref(0);
let imgTimer: number | null = null;

const isMobile = ref(window.innerWidth < MOBILE_BREAKPOINT);
function handleResize() {
  const nowMobile = window.innerWidth < MOBILE_BREAKPOINT;
  if (nowMobile !== isMobile.value) {
    isMobile.value = nowMobile;
    currentIndex.value = 0;
  }
}
window.addEventListener("resize", handleResize);

const activeImages = computed(() =>
  isMobile.value ? randomFive2.value : randomFive.value
);

function startBgTimer() {
  stopBgTimer();
  imgTimer = window.setInterval(() => {
    const len = Math.max(1, activeImages.value.length);
    currentIndex.value = (currentIndex.value + 1) % len;
  }, BG_INTERVAL_MS);
}
function stopBgTimer() {
  if (imgTimer !== null) {
    clearInterval(imgTimer);
    imgTimer = null;
  }
}

// ========== 粒子系统（爱心、连线、数据十字） ==========
const canvasRef = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D;
let animationFrame: number;
let particles: Particle[] = [];
const PARTICLE_COUNT = window.innerWidth < 768 ? 40 : 80;

interface Particle {
  x: number;
  y: number;
  size: number;
  speedX: number;
  speedY: number;
  color: string;
  alpha: number;
  pulse: number;
  glitchOffset: number;
  glitchTimer: number;
  shape: "circle" | "heart" | "cross";
}

function initParticles(width: number, height: number) {
  particles = [];
  for (let i = 0; i < PARTICLE_COUNT; i++) {
    const rand = Math.random();
    let shape: "circle" | "heart" | "cross" = "circle";
    if (rand < 0.2) shape = "heart";
    else if (rand < 0.4) shape = "cross";
    particles.push({
      x: Math.random() * width,
      y: Math.random() * height,
      size: 4 + Math.random() * 16,
      speedX: (Math.random() - 0.5) * 0.3,
      speedY: 0.2 + Math.random() * 1.0,
      color: Math.random() > 0.5 ? "#ff8cb0" : "#6ed4ff",
      alpha: 0.3 + Math.random() * 0.5,
      pulse: 0,
      glitchOffset: 0,
      glitchTimer: Math.random() * 100,
      shape,
    });
  }
}

function updateParticles(dt: number, width: number, height: number) {
  particles.forEach((p) => {
    p.x += p.speedX * dt * 60;
    p.y += p.speedY * dt * 60;

    if (p.y > height + p.size) {
      p.y = -p.size;
      p.x = Math.random() * width;
    }
    if (p.x > width + p.size) p.x = -p.size;
    if (p.x < -p.size) p.x = width + p.size;

    p.glitchTimer += dt;
    if (p.glitchTimer > 0.15) {
      p.glitchOffset = (Math.random() - 0.5) * 10;
      p.glitchTimer = 0;
    } else {
      p.glitchOffset *= 0.9;
    }

    p.pulse = Math.sin(Date.now() * 0.005 + p.x) * 0.2 + 1;
  });
}

function drawParticles(width: number, height: number) {
  ctx.clearRect(0, 0, width, height);

  // 极淡网格
  ctx.strokeStyle = "rgba(255,140,176,0.08)";
  ctx.lineWidth = 0.5;
  for (let i = 0; i < width; i += 40) {
    ctx.beginPath();
    ctx.moveTo(i, 0);
    ctx.lineTo(i, height);
    ctx.stroke();
  }
  for (let i = 0; i < height; i += 40) {
    ctx.beginPath();
    ctx.moveTo(0, i);
    ctx.lineTo(width, i);
    ctx.stroke();
  }

  // 粒子连线
  ctx.lineWidth = 0.5;
  for (let i = 0; i < particles.length; i++) {
    for (let j = i + 1; j < particles.length; j++) {
      const dx =
        particles[i].x +
        particles[i].glitchOffset -
        (particles[j].x + particles[j].glitchOffset);
      const dy = particles[i].y - particles[j].y;
      const dist = Math.sqrt(dx * dx + dy * dy);
      if (dist < 80) {
        ctx.globalAlpha = 0.1 * (1 - dist / 80);
        ctx.strokeStyle = particles[i].color;
        ctx.beginPath();
        ctx.moveTo(particles[i].x + particles[i].glitchOffset, particles[i].y);
        ctx.lineTo(particles[j].x + particles[j].glitchOffset, particles[j].y);
        ctx.stroke();
      }
    }
  }

  // 绘制粒子
  particles.forEach((p) => {
    ctx.save();
    ctx.globalAlpha = p.alpha;
    ctx.shadowColor = p.color;
    ctx.shadowBlur = 12 * p.pulse;
    ctx.fillStyle = p.color;
    ctx.translate(p.x + p.glitchOffset, p.y);
    ctx.scale(p.pulse, p.pulse);

    if (p.shape === "heart") {
      ctx.font = `${p.size}px "Arial", "Segoe UI Emoji"`;
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      ctx.fillText("❤️", 0, 0);
    } else if (p.shape === "cross") {
      ctx.fillRect(-2, -p.size / 2, 4, p.size);
      ctx.fillRect(-p.size / 2, -2, p.size, 4);
    } else {
      ctx.beginPath();
      ctx.arc(0, 0, p.size / 2, 0, Math.PI * 2);
      ctx.fill();
    }
    ctx.restore();
  });
}

let lastTime = 0;
function animate(now: number) {
  if (!lastTime) lastTime = now;
  const dt = Math.min(0.05, (now - lastTime) / 1000);
  lastTime = now;

  const canvas = canvasRef.value!;
  const width = canvas.width / devicePixelRatio;
  const height = canvas.height / devicePixelRatio;

  updateParticles(dt, width, height);
  drawParticles(width, height);

  animationFrame = requestAnimationFrame(animate);
}

function resizeCanvas() {
  const canvas = canvasRef.value!;
  const parent = canvas.parentElement!;
  const dpr = window.devicePixelRatio || 1;
  const w = parent.clientWidth;
  const h = Math.max(parent.clientHeight, 500);

  canvas.style.width = w + "px";
  canvas.style.height = h + "px";
  canvas.width = w * dpr;
  canvas.height = h * dpr;

  ctx = canvas.getContext("2d")!;
  ctx.setTransform(1, 0, 0, 1, 0, 0);
  ctx.scale(dpr, dpr);

  initParticles(w, h);
  lastTime = 0;
}

// ========== 同步率环形 ==========
const syncProgress = ref(0.5);
let syncInterval: number;
function updateSyncRing() {
  syncProgress.value = (Math.sin(Date.now() * 0.001) + 1) / 2;
}
const ringDashoffset = computed(() => 283 * (1 - syncProgress.value));

// ========== 性能优化：页面不可见时暂停 ==========
function handleVisibilityChange() {
  if (document.hidden) {
    cancelAnimationFrame(animationFrame);
    stopBgTimer();
    clearInterval(syncInterval);
    clearInterval(quoteTimer);
  } else {
    animationFrame = requestAnimationFrame(animate);
    startBgTimer();
    syncInterval = window.setInterval(updateSyncRing, 50);
    quoteTimer = window.setInterval(addFloatingQuote, 8000);
  }
}

onMounted(() => {
  typingTimer = window.setTimeout(type, 500);
  resizeCanvas();
  window.addEventListener("resize", resizeCanvas);
  animationFrame = requestAnimationFrame(animate);

  startBgTimer();
  syncInterval = window.setInterval(updateSyncRing, 50);
  quoteTimer = window.setInterval(addFloatingQuote, 8000);
  // 立即添加一个初始台词
  setTimeout(addFloatingQuote, 1000);

  document.addEventListener("visibilitychange", handleVisibilityChange);
});

onBeforeUnmount(() => {
  window.clearTimeout(typingTimer);
  window.removeEventListener("resize", resizeCanvas);
  cancelAnimationFrame(animationFrame);
  stopBgTimer();
  clearInterval(syncInterval);
  clearInterval(quoteTimer);
  document.removeEventListener("visibilitychange", handleVisibilityChange);
});
</script>

<style scoped lang="scss">
/* 爱弥斯风格首页（优化版） */
.aimis-home {
  --pink-core: #ff8cb0;
  --blue-glitch: #6ed4ff;
  --bg-deep: linear-gradient(145deg, #1a1025 0%, #23182b 100%);
  --grid-color: rgba(255, 140, 176, 0.1);
  --heart-glow: rgba(255, 140, 176, 0.5);

  min-height: 100vh;
  background: var(--bg-deep);
  position: relative;
  overflow: hidden;
  color: white;
  font-family: "Inter", "PingFang SC", sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  // 全局网格（带闪烁）
  &::after {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-image: linear-gradient(var(--grid-color) 1px, transparent 1px),
      linear-gradient(90deg, var(--grid-color) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 1;
    opacity: 0.25;
    animation: grid-flicker 4s infinite;
  }

  // 扫描线
  &::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: repeating-linear-gradient(
      0deg,
      rgba(255, 255, 255, 0.02) 0px,
      rgba(255, 255, 255, 0) 2px,
      transparent 4px
    );
    pointer-events: none;
    z-index: 2;
    animation: scan 10s linear infinite;
  }

  // 新增装饰：四角闪烁的数据碎片
  &::after {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: radial-gradient(
        circle at 10% 10%,
        rgba(255, 140, 176, 0.1) 0%,
        transparent 30%
      ),
      radial-gradient(
        circle at 90% 90%,
        rgba(110, 212, 255, 0.1) 0%,
        transparent 30%
      );
    pointer-events: none;
    z-index: 2;
    animation: corner-glow 6s ease-in-out infinite;
  }

  .bg-carousel {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;
    opacity: 0.35;
    filter: hue-rotate(-10deg) saturate(1.2);

    .bg-layer {
      position: absolute;
      inset: 0;
      overflow: hidden;

      .bg-img {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        object-fit: cover;
        display: block;
        opacity: 0;
        transform: scale(1.02);
        transition: opacity 900ms ease, transform 900ms ease;
        filter: brightness(0.68) contrast(0.96) saturate(0.9) hue-rotate(-6deg);
        mix-blend-mode: screen;
      }

      .bg-img.active {
        opacity: 1;
        transform: scale(1);
        filter: brightness(0.92) contrast(1.02) saturate(1.04);
        animation: bg-glitch 8s infinite;
      }
    }
  }

  .particle-canvas {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 3;
    pointer-events: none;
  }

  .hero {
    position: relative;
    z-index: 10;
    text-align: center;
    max-width: 900px;
    padding: 20px;

    .sync-ring {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 180px;
      height: 180px;
      z-index: -1;
      opacity: 0.3;
      pointer-events: none;

      svg {
        width: 100%;
        height: 100%;
        filter: drop-shadow(0 0 10px var(--pink-core));
      }
    }

    .main-title {
      font-family: "Cinzel", "Noto Serif SC", serif;
      font-size: 7rem;
      font-weight: 900;
      line-height: 1;
      margin: 0 0 20px;
      text-transform: uppercase;
      background: linear-gradient(
        135deg,
        var(--pink-core) 20%,
        var(--blue-glitch) 80%
      );
      background-clip: text;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      text-shadow: 3px 3px 0 rgba(110, 212, 255, 0.5),
        -3px -3px 0 var(--pink-core);
      animation: glitch 3s infinite alternate;
      position: relative;

      &::before,
      &::after {
        content: attr(data-text);
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: transparent;
        clip: rect(0, 0, 0, 0);
      }
      &::before {
        left: -3px;
        text-shadow: 3px 0 var(--blue-glitch);
        animation: glitch-before 0.4s infinite linear alternate-reverse;
      }
      &::after {
        left: 3px;
        text-shadow: -3px 0 var(--pink-core);
        animation: glitch-after 0.3s infinite linear alternate-reverse;
      }
    }

    .subtitle {
      font-size: 2rem;
      font-family: "Dancing Script", cursive;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 10px;
      color: rgba(255, 255, 255, 0.95);
      text-shadow: 0 0 12px var(--heart-glow);
      min-height: 3rem;

      .typed {
        display: inline-block;
      }
      .cursor {
        width: 8px;
        height: 1.2em;
        background: linear-gradient(
          180deg,
          var(--blue-glitch),
          var(--pink-core)
        );
        border-radius: 2px;
        animation: blink 1s steps(1) infinite;
      }
    }

    .heart-float {
      position: absolute;
      bottom: -30px;
      right: -20px;
      font-size: 4rem;
      animation: float-heart 4s ease-in-out infinite;
      filter: drop-shadow(0 0 20px var(--heart-glow));
      opacity: 0.8;
    }
  }

  // 悬浮语录
  .floating-quote {
    position: absolute;
    z-index: 15;
    background: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(8px);
    border: 1px solid var(--pink-core);
    border-radius: 30px;
    padding: 8px 20px;
    color: white;
    font-size: 1rem;
    font-family: "Noto Serif SC", serif;
    white-space: nowrap;
    box-shadow: 0 0 20px var(--heart-glow);
    animation: quote-appear 0.3s ease-out, quote-float 3s ease-in-out infinite;
    pointer-events: none;
  }

  // 探索星炬按钮
  .enter-btn {
    position: absolute;
    bottom: 100px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10;
    background: radial-gradient(
      circle at 30% 30%,
      var(--pink-core),
      var(--blue-glitch)
    );
    border: none;
    border-radius: 50px;
    padding: 14px 48px;
    color: white;
    font-size: 1.3rem;
    font-weight: 600;
    letter-spacing: 2px;
    text-decoration: none;
    box-shadow: 0 0 30px var(--heart-glow), 0 6px 14px rgba(0, 0, 0, 0.5);
    transition: all 0.3s;
    animation: btn-pulse 2s infinite,
      cursorAnimation_link 1s infinite step-start;

    &:hover {
      transform: translateX(-50%) scale(1.05);
      box-shadow: 0 0 50px var(--pink-core);
    }
  }

  .footer {
    position: absolute;
    bottom: 20px;
    left: 0;
    right: 0;
    z-index: 10;
    text-align: center;
    color: rgba(255, 255, 255, 0.7);
    font-size: 0.9rem;
    backdrop-filter: blur(2px);
    padding: 10px;

    .slogan {
      background: linear-gradient(90deg, var(--pink-core), var(--blue-glitch));
      background-clip: text;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      font-weight: 600;
      margin-bottom: 5px;
    }
    .copyright {
      font-size: 0.8rem;
      opacity: 0.6;
    }
  }
}

/* 动画定义 */
@keyframes scan {
  0% {
    background-position: 0 0;
  }
  100% {
    background-position: 0 100%;
  }
}
@keyframes grid-flicker {
  0% {
    opacity: 0.2;
  }
  50% {
    opacity: 0.3;
  }
  100% {
    opacity: 0.2;
  }
}
@keyframes corner-glow {
  0% {
    opacity: 0.2;
  }
  50% {
    opacity: 0.4;
  }
  100% {
    opacity: 0.2;
  }
}
@keyframes bg-glitch {
  0%,
  100% {
    filter: brightness(0.92) contrast(1.02) saturate(1.04);
  }
  95% {
    filter: brightness(0.92) contrast(1.02) saturate(1.04) hue-rotate(2deg);
  }
  96% {
    filter: brightness(1.1) contrast(1.1) saturate(1.2) hue-rotate(-2deg);
    transform: translateX(2px);
  }
  97% {
    filter: brightness(0.9) contrast(0.98) saturate(0.9) hue-rotate(2deg);
    transform: translateX(-2px);
  }
}
@keyframes glitch {
  0% {
    text-shadow: 3px 3px 0 rgba(110, 212, 255, 0.5),
      -3px -3px 0 var(--pink-core);
  }
  25% {
    text-shadow: -4px 2px 0 var(--blue-glitch), 4px -2px 0 var(--pink-core);
  }
  50% {
    text-shadow: 3px -3px 0 var(--blue-glitch), -3px 3px 0 var(--pink-core);
  }
  75% {
    text-shadow: 4px 3px 0 var(--pink-core), -4px -3px 0 var(--blue-glitch);
  }
  100% {
    text-shadow: -3px 4px 0 var(--pink-core), 3px -4px 0 var(--blue-glitch);
  }
}
@keyframes glitch-before {
  0% {
    clip: rect(44px, 9999px, 56px, 0);
  }
  100% {
    clip: rect(12px, 9999px, 24px, 0);
  }
}
@keyframes glitch-after {
  0% {
    clip: rect(86px, 9999px, 98px, 0);
  }
  100% {
    clip: rect(38px, 9999px, 52px, 0);
  }
}
@keyframes blink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}
@keyframes float-heart {
  0% {
    transform: translateY(0) rotate(0deg);
  }
  50% {
    transform: translateY(-20px) rotate(10deg);
  }
  100% {
    transform: translateY(0) rotate(0deg);
  }
}
@keyframes quote-appear {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
@keyframes quote-float {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
  100% {
    transform: translateY(0);
  }
}
@keyframes btn-pulse {
  0% {
    box-shadow: 0 0 20px var(--heart-glow), 0 6px 14px rgba(0, 0, 0, 0.5);
  }
  50% {
    box-shadow: 0 0 40px var(--heart-glow), 0 8px 20px rgba(0, 0, 0, 0.6);
  }
  100% {
    box-shadow: 0 0 20px var(--heart-glow), 0 6px 14px rgba(0, 0, 0, 0.5);
  }
}

/* 移动端适配 */
@media (max-width: 768px) {
  .aimis-home {
    .hero .main-title {
      font-size: 4rem;
    }
    .hero .subtitle {
      font-size: 1.5rem;
    }
    .heart-float {
      font-size: 2.5rem;
      bottom: -10px;
      right: 0;
    }
    .enter-btn {
      bottom: 80px;
      padding: 10px 30px;
      font-size: 1.1rem;
    }
    .floating-quote {
      font-size: 0.9rem;
      white-space: normal;
      max-width: 200px;
    }
  }
}
</style>
