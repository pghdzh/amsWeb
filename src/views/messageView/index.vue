<template>
  <div class="aimis-message-board">
    <!-- 背景轮播（绝对定位） -->
    <div class="carousel carousel1" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <div class="carousel carousel2" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive2"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>

    <!-- Canvas粒子背景 -->
    <canvas ref="canvasRef" class="particle-canvas"></canvas>

    <!-- 主内容区（固定高度滚动） -->
    <div class="content-wrapper">
      <!-- 顶部标题 -->
      <header class="board-header">
        <h1 class="glitch" data-text="心迹留言 · 星炬回响">
          心迹留言 · 星炬回响
        </h1>
        <span class="title-count">共{{ count }}条留言</span>
        <p class="subtitle">电子幽灵，心之所向</p>
      </header>

      <!-- 留言列表（滚动） -->
      <section class="message-list" ref="listContainer">
        <transition-group name="msg" tag="div" class="message-list-inner">
          <div v-if="loading" class="skeleton-wrap" key="skeleton">
            <div class="skeleton" v-for="i in 3" :key="i">
              <div class="sk-avatar"></div>
              <div class="sk-lines">
                <div class="sk-line short"></div>
                <div class="sk-line"></div>
              </div>
            </div>
          </div>
          <div
            v-for="(msg, idx) in messages"
            :key="msg.id || msg._tempId || idx"
            class="message-card"
            :class="{
              invaded: msg.isInvaded,
              'invade-flash': msg.isInvaded && msg.flash,
              'typing-active': msg.typing,
            }"
            :data-index="idx"
            ref="messageCards"
          >
            <div class="message-meta">
              <div class="left-meta">
                <div
                  class="name-avatar"
                  :style="{
                    background: msg.isInvaded
                      ? 'linear-gradient(135deg, #ff8cb0, #6ed4ff)'
                      : '',
                  }"
                >
                  {{ msg.isInvaded ? "爱" : getInitial(msg.name) }}
                </div>
                <div class="meta-texts">
                  <div class="message-name">
                    {{ msg.isInvaded ? "爱弥斯" : msg.name || "匿名" }}
                    <span v-if="msg.isInvaded" class="invade-tag">❤️</span>
                  </div>
                  <div class="message-time">
                    {{ formatTime(msg.created_at) }}
                  </div>
                </div>
              </div>
            </div>
            <p
              class="message-content"
              :class="{ 'invaded-text': msg.isInvaded }"
            >
              {{ msg.content }}
              <span v-if="msg.typing" class="typing-cursor"></span>
            </p>
          </div>
        </transition-group>
      </section>

      <!-- 底部留言表单 -->
      <section class="message-form">
        <input
          v-model="name"
          type="text"
          placeholder="你的昵称"
          @keydown.enter.prevent
        />
        <textarea
          v-model="content"
          placeholder="写下你的留言..."
          @keydown.ctrl.enter.prevent="submitMessage"
          @input="autoGrow"
          ref="textareaRef"
        />
        <div class="form-row">
          <div class="hint">按 <kbd>Ctrl</kbd> + <kbd>Enter</kbd> 发送</div>
          <button
            @click="submitMessage"
            :disabled="isSending || !content.trim()"
          >
            {{ isSending ? "发送中..." : "发送" }}
          </button>
        </div>
      </section>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, onBeforeUnmount } from "vue";
import { getMessageList, createMessage } from "@/api/modules/message";

// 数据
const messages = ref<any[]>([]);
const count = ref(0);
const name = ref(localStorage.getItem("message_name") || "");
const content = ref("");
const loading = ref(true);
const isSending = ref(false);
const textareaRef = ref<HTMLTextAreaElement | null>(null);
const listContainer = ref<HTMLElement | null>(null);
const messageCards = ref<HTMLElement[]>([]);

// 入侵相关
const INVASION_DELAY = 10000; // 30秒
let invasionTimer: number | null = null;
let typingInterval: number | null = null; // 打字定时器

// 爱弥斯语录
const aimisQuotes = [
  "但愿我会让你感到骄傲，但愿我没有让你失望",
  "电子幽灵来访了，你的留言我收到了",
  "在数据世界里，为你唱响这首歌",
  "同步率上升……留言已接收",
  "粉发少女的内心，藏着无尽的温柔",
  "星炬的光，永不熄灭",
  "故障中……信号不稳定，但心意不变",
  "你听到远航星的告别了吗？",
  "我是爱弥斯，星炬学院的适格者",
];

// 入侵函数（带打字效果）
const invadeMessage = () => {
  if (!messages.value.length) return;
  // 清除之前的打字定时器
  if (typingInterval) clearInterval(typingInterval);

  const randomIndex = Math.floor(Math.random() * messages.value.length);
  const targetMsg = messages.value[randomIndex];
  const targetCard = document.querySelectorAll(".message-card")[
    randomIndex
  ] as HTMLElement;

  if (!targetCard) return;

  // 1. 滚动到卡片
  targetCard.scrollIntoView({ behavior: "smooth", block: "center" });

  // 2. 闪烁动画（原内容闪烁）
  targetMsg.flash = true; // 触发闪烁类

  // 随机选择语录
  const fullQuote = aimisQuotes[Math.floor(Math.random() * aimisQuotes.length)];
  // 准备打字：清空原内容，设置昵称和标记
  targetMsg.name = "爱弥斯";
  targetMsg.content = "";
  targetMsg.isInvaded = true;
  targetMsg.typing = true; // 显示打字光标

  let charIndex = 0;
  typingInterval = window.setInterval(() => {
    if (charIndex < fullQuote.length) {
      targetMsg.content = fullQuote.slice(0, charIndex + 1);
      charIndex++;
    } else {
      // 打字完成
      clearInterval(typingInterval!);
      typingInterval = null;
      targetMsg.typing = false; // 移除光标
    }
  }, 200); // 每80ms打一字
};

// ========== 获取留言 ==========
const fetchMessages = async () => {
  loading.value = true;
  try {
    const res = await getMessageList({ page: 1, pageSize: 999 });
    messages.value = (res.data || []).map((msg: any) => ({
      ...msg,
      isInvaded: false,
      flash: false,
      typing: false,
    }));
    count.value = res.pagination.total;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// ========== 提交留言 ==========
const submitMessage = async () => {
  if (!content.value.trim() || isSending.value) return;
  isSending.value = true;
  const payload = { name: name.value || "匿名", content: content.value };
  try {
    localStorage.setItem("message_name", name.value);
    content.value = "";
    await createMessage(payload);
    await fetchMessages();
    await nextTick();
    autoGrow();
  } catch (err) {
    console.error(err);
  } finally {
    isSending.value = false;
  }
};

// ========== 工具函数 ==========
const formatTime = (time: string) => {
  if (!time) return "";
  const d = new Date(time);
  return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(
    2,
    "0"
  )}-${String(d.getDate()).padStart(2, "0")} ${String(d.getHours()).padStart(
    2,
    "0"
  )}:${String(d.getMinutes()).padStart(2, "0")}`;
};

const getInitial = (n?: string) =>
  n ? n.trim().slice(0, 1).toUpperCase() : "匿";

const autoGrow = () => {
  const ta = textareaRef.value;
  if (!ta) return;
  ta.style.height = "auto";
  ta.style.height = Math.min(ta.scrollHeight, 220) + "px";
};

// ========== 背景轮播 ==========
const modules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs: string[] = Object.values(modules).map((mod: any) => mod.default);
const modules2 = import.meta.glob("@/assets/images2/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs2: string[] = Object.values(modules2).map(
  (mod: any) => mod.default
);

function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
const randomFive = ref<string[]>(shuffle(allSrcs).slice(0, 5));
const randomFive2 = ref<string[]>(shuffle(allSrcs2).slice(0, 5));
const currentIndex = ref(0);
let imgTimer: number | undefined;

// ========== Canvas 粒子 ==========
const canvasRef = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D;
let animationFrame: number;
let particles: Particle[] = [];
const PARTICLE_COUNT = window.innerWidth < 768 ? 30 : 60;

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
    } else p.glitchOffset *= 0.9;
    p.pulse = Math.sin(Date.now() * 0.005 + p.x) * 0.2 + 1;
  });
}

function drawParticles(width: number, height: number) {
  ctx.clearRect(0, 0, width, height);
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
  // 连线
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
  // 画粒子
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
  const h = parent.clientHeight;
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

// ========== 生命周期 ==========
onMounted(() => {
  fetchMessages();
  imgTimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  nextTick(() => autoGrow());

  resizeCanvas();
  window.addEventListener("resize", resizeCanvas);
  animationFrame = requestAnimationFrame(animate);

  invasionTimer = window.setTimeout(invadeMessage, INVASION_DELAY);
});

onBeforeUnmount(() => {
  if (imgTimer) clearInterval(imgTimer);
  if (invasionTimer) clearTimeout(invasionTimer);
  if (typingInterval) clearInterval(typingInterval);
  window.removeEventListener("resize", resizeCanvas);
  cancelAnimationFrame(animationFrame);
});
</script>

<style scoped lang="scss">
.aimis-message-board {
  --pink-core: #ff8cb0;
  --blue-glitch: #6ed4ff;
  --bg-deep: linear-gradient(145deg, #1a1025 0%, #23182b 100%);
  --grid-color: rgba(255, 140, 176, 0.1);
  --heart-glow: rgba(255, 140, 176, 0.5);
  padding-top: 80px;
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: var(--bg-deep);
  color: white;
  font-family: "Noto Sans SC", sans-serif;

  // 科技网格（背景层）
  &::after {
    content: "";
    position: absolute;
    inset: 0;
    background-image: linear-gradient(var(--grid-color) 1px, transparent 1px),
      linear-gradient(90deg, var(--grid-color) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
    opacity: 0.25;
    animation: grid-flicker 4s infinite;
  }

  // 扫描线
  &::before {
    content: "";
    position: absolute;
    inset: 0;
    background: repeating-linear-gradient(
      0deg,
      rgba(255, 255, 255, 0.02) 0px,
      rgba(255, 255, 255, 0) 2px,
      transparent 4px
    );
    pointer-events: none;
    z-index: 0;
    animation: scan 10s linear infinite;
  }

  .carousel {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;
    opacity: 0.3;
    filter: hue-rotate(-10deg) saturate(1.2);

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 1s ease;
      filter: brightness(0.68) contrast(0.96) saturate(0.9) hue-rotate(-6deg);
      &.active {
        opacity: 1;
      }
    }
  }
  .carousel2 {
    display: none;
  }

  .particle-canvas {
    position: absolute;
    inset: 0;
    z-index: 1;
    pointer-events: none;
  }

  .content-wrapper {
    position: relative;
    z-index: 2;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    padding: 20px;
    box-sizing: border-box;
    max-width: 1000px;
    margin: 0 auto;
  }

  .board-header {
    flex-shrink: 0;
    display: flex;
    align-items: baseline;
    gap: 15px;
    padding: 15px 20px;
    background: rgba(0, 0, 0, 0.2);
    backdrop-filter: blur(10px);
    border: 1px solid rgba(255, 140, 176, 0.3);
    border-radius: 40px;
    margin-bottom: 20px;
    box-shadow: 0 0 30px var(--heart-glow);

    h1 {
      margin: 0;
      font-family: "Cinzel", serif;
      font-size: 1.8rem;
      background: linear-gradient(
        135deg,
        var(--pink-core) 20%,
        var(--blue-glitch) 80%
      );
      background-clip: text;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      text-shadow: 2px 2px 0 rgba(110, 212, 255, 0.5);
    }

    .title-count {
      color: var(--pink-light);
      font-size: 0.9rem;
    }

    .subtitle {
      margin-left: auto;
      color: var(--blue-glitch);
      font-style: italic;
    }
  }

  .message-list {
    flex: 1 1 auto;
    overflow-y: auto;
    margin-bottom: 20px;
    padding-right: 5px;

    &::-webkit-scrollbar {
      width: 6px;
    }
    &::-webkit-scrollbar-track {
      background: rgba(0, 0, 0, 0.1);
    }
    &::-webkit-scrollbar-thumb {
      background: var(--pink-core);
      border-radius: 10px;
    }

    .message-list-inner {
      display: flex;
      flex-direction: column;
      gap: 16px;
    }

    .skeleton-wrap {
      .skeleton {
        display: flex;
        gap: 12px;
        padding: 15px;
        background: rgba(0, 0, 0, 0.2);
        border-radius: 12px;
      }
      .sk-avatar {
        width: 44px;
        height: 44px;
        border-radius: 10px;
        background: linear-gradient(
          90deg,
          var(--pink-core),
          var(--blue-glitch)
        );
      }
      .sk-lines {
        flex: 1;
        .sk-line {
          height: 10px;
          border-radius: 6px;
          background: rgba(255, 255, 255, 0.05);
          margin-bottom: 8px;
        }
        .sk-line.short {
          width: 40%;
        }
      }
    }
  }

  .message-card {
    background: rgba(0, 0, 0, 0.3);
    backdrop-filter: blur(8px);
    border: 1px solid rgba(255, 140, 176, 0.2);
    border-radius: 20px;
    padding: 15px 20px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
    transition: transform 0.2s, border-color 0.2s;

    &:hover {
      transform: translateY(-3px);
      border-color: var(--pink-core);
    }

    &.invaded {
      border-color: var(--pink-core);
      box-shadow: 0 0 20px var(--pink-core);
    }

    &.invade-flash {
      animation: flash-glitch 1s ease;
    }

    &.typing-active .message-content {
      border-right: 2px solid var(--pink-core);
    }

    .message-meta {
      display: flex;
      justify-content: space-between;
      margin-bottom: 10px;

      .left-meta {
        display: flex;
        gap: 12px;
        align-items: center;

        .name-avatar {
          width: 48px;
          height: 48px;
          border-radius: 10px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-weight: 900;
          font-size: 18px;
          background: linear-gradient(
            135deg,
            var(--pink-core),
            var(--blue-glitch)
          );
          box-shadow: 0 0 10px var(--heart-glow);
        }

        .meta-texts {
          .message-name {
            font-size: 16px;
            font-weight: 700;
            display: flex;
            align-items: center;
            gap: 5px;
            .invade-tag {
              font-size: 18px;
              animation: pulse-heart 1.5s infinite;
            }
          }
          .message-time {
            font-size: 12px;
            color: rgba(255, 255, 255, 0.5);
          }
        }
      }
    }

    .message-content {
      margin: 0;
      font-size: 15px;
      line-height: 1.6;
      word-break: break-word;
      &.invaded-text {
        color: var(--pink-light);
        text-shadow: 0 0 5px var(--pink-core);
      }
    }
  }

  .message-form {
    flex-shrink: 0;
    background: rgba(0, 0, 0, 0.3);
    backdrop-filter: blur(10px);
    border: 1px solid rgba(255, 140, 176, 0.3);
    border-radius: 30px;
    padding: 15px 20px;
    box-shadow: 0 0 30px var(--heart-glow);

    input,
    textarea {
      width: 100%;
      background: rgba(0, 0, 0, 0.2);
      border: 1px solid rgba(255, 140, 176, 0.2);
      border-radius: 20px;
      padding: 10px 15px;
      color: white;
      font-size: 14px;
      outline: none;
      transition: border-color 0.2s, box-shadow 0.2s;

      &:focus {
        border-color: var(--pink-core);
        box-shadow: 0 0 10px var(--pink-core);
      }
      &::placeholder {
        color: rgba(255, 255, 255, 0.3);
      }
    }

    textarea {
      min-height: 60px;
      max-height: 150px;
      resize: none;
      margin-top: 10px;
    }

    .form-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-top: 10px;

      .hint {
        color: rgba(255, 255, 255, 0.6);
        font-size: 13px;
        kbd {
          background: rgba(0, 0, 0, 0.5);
          border-radius: 4px;
          padding: 2px 6px;
          color: var(--pink-light);
        }
      }

      button {
        background: linear-gradient(
          135deg,
          var(--pink-core),
          var(--blue-glitch)
        );
        border: none;
        border-radius: 30px;
        padding: 8px 25px;
        color: white;
        font-weight: 600;
        animation: cursorAnimation_link 1s infinite step-start;
        box-shadow: 0 0 15px var(--heart-glow);
        transition: transform 0.2s, box-shadow 0.2s;

        &:hover:not(:disabled) {
          transform: scale(1.05);
          box-shadow: 0 0 25px var(--pink-core);
        }
        &:disabled {
          opacity: 0.5;
          animation: cursorAnimation_disabled 1s infinite step-start;
        }
      }
    }
  }

  /* 移动端适配 */
  @media (max-width: 768px) {
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }
    .board-header {
      flex-wrap: wrap;
      h1 {
        font-size: 1.4rem;
      }
      .subtitle {
        margin-left: 0;
      }
    }
  }
}

/* 动画 */
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
@keyframes flash-glitch {
  0% {
    box-shadow: 0 0 0 transparent;
    border-color: transparent;
    background: rgba(0, 0, 0, 0.3);
  }
  20% {
    box-shadow: 0 0 30px var(--pink-core);
    border-color: var(--pink-core);
    background: rgba(255, 140, 176, 0.2);
    transform: translateX(2px);
  }
  40% {
    box-shadow: 0 0 30px var(--blue-glitch);
    border-color: var(--blue-glitch);
    background: rgba(110, 212, 255, 0.2);
    transform: translateX(-2px);
  }
  60% {
    box-shadow: 0 0 30px var(--pink-core);
    border-color: var(--pink-core);
    background: rgba(255, 140, 176, 0.2);
    transform: translateX(2px);
  }
  80% {
    box-shadow: 0 0 30px var(--blue-glitch);
    border-color: var(--blue-glitch);
    background: rgba(110, 212, 255, 0.2);
    transform: translateX(-2px);
  }
  100% {
    box-shadow: 0 0 0 transparent;
    border-color: var(--pink-core);
    background: rgba(0, 0, 0, 0.3);
    transform: translateX(0);
  }
}
@keyframes pulse-heart {
  0% {
    transform: scale(1);
    opacity: 0.8;
  }
  50% {
    transform: scale(1.2);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 0.8;
  }
}
</style>
