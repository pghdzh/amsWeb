<template>
  <div class="ack-page">
    <!-- 故障光效背景层 + 数据粒子 + 扫描线 -->
    <div class="glitch-bg"></div>
    <div class="data-particles"></div>
    <div class="scan-line"></div>

    <!-- 核心能量状态 -->
    <div class="core-status">
      <span class="pulse-dot"></span>
      <span class="status-text">AI-EMS CORE: 98%</span>
    </div>

    <div class="ack-container">
      <h1 class="page-title glitch-text" data-text="鸣谢名单">鸣谢名单</h1>
      <ul class="ack-list">
        <li
          v-for="(item, index) in acknowledgements"
          :key="index"
          class="ack-item"
          :style="{ animationDelay: index * 0.15 + 's' }"
        >
          <div class="ack-card">
            <!-- 故障边框 -->
            <div class="glitch-frame"></div>
            <!-- 全息扫描线 (hover时出现) -->
            <div class="hologram-scan"></div>

            <span class="nickname">{{ item.nickname }}:</span>
            <span class="suggestion">{{ item.suggestion }}</span>
          </div>
        </li>
      </ul>
      <div v-if="acknowledgements.length === 0" class="empty">
        > 暂无鸣谢内容_
      </div>
    </div>

    <footer class="ack-footer">
      <span>> © 2025 爱弥斯电子设定集 · 感谢所有支持者_</span>
    </footer>

    <!-- 浮动精灵（少量） -->
    <div class="floating-sprites">
      <div
        v-for="(sprite, i) in spriteList"
        :key="i"
        class="sprite"
        :style="{ top: sprite.top + 'px', left: sprite.left + 'px' }"
      ></div>
    </div>

    <!-- 爱弥斯核心语录 -->
    <div class="ai-message" v-if="showAiMessage" @click="nextMessage">
      <span class="msg-label">爱弥斯说:</span>
      <span class="msg-content">{{ currentMessage }}</span>
      <span class="msg-cursor">_</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, onMounted, onBeforeUnmount, nextTick } from "vue";
import { gsap } from "gsap";

const acknowledgements = reactive([
  {
    nickname: "小笨12138",
    suggestion: "鼠标改成像素小爱",
  },
  {
    nickname: "擢阳_9tivk",
    suggestion: "给爱弥斯的名字加一个数据化特效",
  },
  {
    nickname: "老广东響爷",
    suggestion: "飞行雪绒漂浮",
  },
  {
    nickname: "HNV-南十字星",
    suggestion: "BGM用星炬不熄的纯音乐版本(这个是几首随机)",
  },
  {
    nickname: "赤雪",
    suggestion:
      "放音乐的图标想换个像素小爱带耳机(赤雪提供了图片但是我没实现动态效果就还是没采用)",
  },
]);

// 爱弥斯语录
const messages = [
  "感谢每一位支持者...",
  "你们的建议我都记在心里。",
  "风会记住每一份心意。",
  "故障率低于阈值，系统正常。",
];
const currentMessage = ref(messages[0]);
const showAiMessage = ref(true);
let msgInterval: number;

function nextMessage() {
  const next = (messages.indexOf(currentMessage.value) + 1) % messages.length;
  currentMessage.value = messages[next];
}

// 浮动精灵
interface Sprite {
  top: number;
  left: number;
}
const spriteList = ref<Sprite[]>([]);

onMounted(() => {
  // 初始化语录轮换
  msgInterval = window.setInterval(() => {
    nextMessage();
  }, 8000);

  // 初始化浮动精灵（数量少一点）
  const spriteCount = 4;
  const spriteSize = 30;
  spriteList.value = [];
  for (let i = 0; i < spriteCount; i++) {
    spriteList.value.push({
      left: Math.random() * (window.innerWidth - spriteSize),
      top: Math.random() * (window.innerHeight - spriteSize),
    });
  }
  nextTick(() => {
    const sprites = document.querySelectorAll<HTMLElement>(".sprite");
    sprites.forEach((el, idx) => {
      gsap.fromTo(
        el,
        { opacity: 0, scale: 0 },
        { opacity: 0.4, scale: 1, duration: 1, delay: idx * 0.1 }
      );
      gsap.to(el, {
        y: "+=20",
        x: "+=10",
        duration: 5 + Math.random() * 2,
        yoyo: true,
        repeat: -1,
        ease: "sine.inOut",
        modifiers: {
          x: (x) => {
            let val = parseFloat(x);
            const maxX = window.innerWidth - 30;
            if (val < 0) val = 0;
            if (val > maxX) val = maxX;
            return val + "px";
          },
          y: (y) => {
            let val = parseFloat(y);
            const maxY = window.innerHeight - 30;
            if (val < 0) val = 0;
            if (val > maxY) val = maxY;
            return val + "px";
          },
        },
      });
      el.addEventListener("mouseenter", () => {
        gsap.killTweensOf(el);
        gsap.to(el, {
          x: "+=" + (Math.random() - 0.5) * 150,
          y: "+=" + (Math.random() - 0.5) * 150,
          duration: 0.6,
          ease: "back.out(2)",
        });
      });
    });
  });
});

onBeforeUnmount(() => {
  if (msgInterval) clearInterval(msgInterval);
});
</script>

<style scoped lang="scss">
/* ===== 爱弥斯风格配色系统 ===== */

/* ===== 基础动画 ===== */
@keyframes glitch {
  0% {
    transform: translate(0);
    text-shadow: -2px 0 var(--blue-glitch), 2px 2px var(--pink-core);
  }
  20% {
    transform: translate(-2px, 2px);
    text-shadow: 2px -2px var(--blue-glitch), -2px 2px var(--pink-core);
  }
  40% {
    transform: translate(2px, -2px);
    text-shadow: -2px 2px var(--blue-glitch), 2px -2px var(--pink-core);
  }
  60% {
    transform: translate(0);
    text-shadow: 2px 0 var(--blue-glitch), -2px -2px var(--pink-core);
  }
  80% {
    transform: translate(2px, 2px);
    text-shadow: -2px -2px var(--blue-glitch), 2px 0 var(--pink-core);
  }
  100% {
    transform: translate(0);
    text-shadow: none;
  }
}
@keyframes pulse-glow {
  0%,
  100% {
    filter: drop-shadow(0 0 5px var(--pink-glow))
      drop-shadow(0 0 10px var(--blue-glitch));
  }
  50% {
    filter: drop-shadow(0 0 15px var(--pink-core))
      drop-shadow(0 0 20px var(--blue-glow));
  }
}
@keyframes scanline {
  0% {
    transform: translateY(-100%);
  }
  100% {
    transform: translateY(100%);
  }
}
@keyframes blink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.3;
  }
}
@keyframes float-particle {
  0% {
    transform: translate(0, 0) rotate(0deg);
    opacity: 0;
  }
  10% {
    opacity: 0.5;
  }
  90% {
    opacity: 0.5;
  }
  100% {
    transform: translate(calc(100vw * var(--dx)), calc(100vh * var(--dy)))
      rotate(360deg);
    opacity: 0;
  }
}
@keyframes hologram-scan {
  0% {
    top: -10%;
    opacity: 0;
  }
  20% {
    opacity: 0.8;
  }
  80% {
    opacity: 0.8;
  }
  100% {
    top: 110%;
    opacity: 0;
  }
}
@keyframes core-pulse {
  0% {
    box-shadow: 0 0 5px var(--pink-core), 0 0 15px var(--blue-glitch);
  }
  50% {
    box-shadow: 0 0 15px var(--pink-core), 0 0 30px var(--blue-glow),
      0 0 45px var(--pink-light);
  }
  100% {
    box-shadow: 0 0 5px var(--pink-core), 0 0 15px var(--blue-glitch);
  }
}
@keyframes borderRotate {
  0% {
    filter: hue-rotate(0deg);
  }
  100% {
    filter: hue-rotate(360deg);
  }
}
@keyframes itemIn {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* ===== 全局样式 ===== */
.ack-page {
  --pink-core: #ff8cb0;
  --pink-light: #ffb3cc;
  --pink-glow: #ffb6d9;
  --blue-glitch: #6ed4ff;
  --blue-glow: #9ae2ff;
  --purple-mid: #e0a0ff;
  --white-glow: #fbefff;
  --bg-deep: linear-gradient(145deg, #1a1028 0%, #281e30 100%);
  --grid-color: rgba(255, 140, 176, 0.15);
  --glitch-shadow: rgba(110, 212, 255, 0.5);
  --heart-glow: rgba(255, 140, 176, 0.7);
  min-height: 100vh;
  padding-top: 64px;
  background: var(--bg-deep);
  font-family: "JetBrains Mono", "Fira Code", "PingFang SC", monospace;
  position: relative;
  overflow-x: hidden;
  display: flex;
  flex-direction: column;
}

/* 故障背景层 */
.glitch-bg {
  position: fixed;
  inset: 0;
  background: radial-gradient(
      circle at 30% 40%,
      rgba(255, 140, 176, 0.1) 0%,
      transparent 30%
    ),
    radial-gradient(
      circle at 70% 60%,
      rgba(110, 212, 255, 0.08) 0%,
      transparent 35%
    ),
    repeating-linear-gradient(
      45deg,
      transparent 0px,
      transparent 10px,
      rgba(255, 140, 176, 0.03) 10px,
      rgba(110, 212, 255, 0.03) 11px,
      transparent 11px,
      transparent 20px
    );
  pointer-events: none;
  z-index: 0;
  opacity: 0.8;
  &::before {
    content: "";
    position: absolute;
    inset: 0;
    background-image: linear-gradient(var(--grid-color) 1px, transparent 1px),
      linear-gradient(90deg, var(--grid-color) 1px, transparent 1px);
    background-size: 40px 40px;
    mask-image: radial-gradient(circle at 50% 50%, black 40%, transparent 70%);
  }
  &::after {
    content: "";
    position: absolute;
    inset: 0;
    background: repeating-linear-gradient(
      0deg,
      rgba(0, 0, 0, 0) 0px,
      rgba(255, 140, 176, 0.05) 2px,
      rgba(0, 0, 0, 0) 4px
    );
    pointer-events: none;
    animation: scanline 12s linear infinite;
  }
}

/* 数据粒子层 */
.data-particles {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 1;
  overflow: hidden;
  &::before {
    content: "";
    position: absolute;
    width: 100%;
    height: 100%;
    background-image: radial-gradient(
        circle at 10% 20%,
        var(--pink-core) 1px,
        transparent 1px
      ),
      radial-gradient(
        circle at 30% 70%,
        var(--blue-glitch) 1px,
        transparent 1px
      ),
      radial-gradient(circle at 50% 40%, var(--purple-mid) 2px, transparent 2px),
      radial-gradient(circle at 80% 60%, var(--pink-light) 1px, transparent 1px),
      radial-gradient(circle at 90% 15%, var(--blue-glow) 1px, transparent 1px);
    background-size: 200px 200px;
    animation: float-particle 20s linear infinite;
    opacity: 0.3;
  }
}

/* 核心能量状态 */
.core-status {
  position: fixed;
  top: 20px;
  left: 20px;
  z-index: 10;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 12px;
  background: rgba(26, 16, 40, 0.6);
  backdrop-filter: blur(5px);
  border: 1px solid var(--pink-core);
  border-radius: 30px;
  font-size: 0.8rem;
  color: var(--white-glow);
  box-shadow: 0 0 15px var(--blue-glitch);
  .pulse-dot {
    width: 8px;
    height: 8px;
    background: var(--pink-core);
    border-radius: 50%;
    animation: core-pulse 1.5s infinite;
  }
  .status-text {
    letter-spacing: 1px;
  }
}

/* 主容器 */
.ack-container {
  flex: 1;
  padding: 2rem;
  max-width: 700px;
  margin: 0 auto;
  position: relative;
  z-index: 5;
  color: var(--white-glow);
  width: 100%;
  box-sizing: border-box;
}

/* 标题 */
.page-title {
  font-size: 2.2rem;
  text-align: center;
  margin-bottom: 2rem;
  color: var(--pink-core);
  text-shadow: 0 0 10px var(--pink-core), 0 0 20px var(--blue-glitch);
  &.glitch-text {
    animation: glitch 3s infinite;
    position: relative;
    &::before,
    &::after {
      content: attr(data-text);
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
    &::before {
      left: 2px;
      text-shadow: -2px 0 var(--blue-glitch);
      clip: rect(44px, 450px, 56px, 0);
      animation: glitch-anim 5s infinite linear alternate-reverse;
    }
    &::after {
      left: -2px;
      text-shadow: -2px 0 var(--pink-core);
      clip: rect(44px, 450px, 56px, 0);
      animation: glitch-anim2 5s infinite linear alternate-reverse;
    }
  }
}
@keyframes glitch-anim {
  0% {
    clip: rect(31px, 9999px, 94px, 0);
  }
  5% {
    clip: rect(70px, 9999px, 71px, 0);
  }
  10% {
    clip: rect(29px, 9999px, 83px, 0);
  }
  15% {
    clip: rect(16px, 9999px, 91px, 0);
  }
  20% {
    clip: rect(2px, 9999px, 36px, 0);
  }
  25% {
    clip: rect(27px, 9999px, 9px, 0);
  }
  30% {
    clip: rect(9px, 9999px, 53px, 0);
  }
  35% {
    clip: rect(17px, 9999px, 24px, 0);
  }
  40% {
    clip: rect(74px, 9999px, 61px, 0);
  }
  45% {
    clip: rect(17px, 9999px, 83px, 0);
  }
  50% {
    clip: rect(74px, 9999px, 55px, 0);
  }
  55% {
    clip: rect(38px, 9999px, 48px, 0);
  }
  60% {
    clip: rect(94px, 9999px, 42px, 0);
  }
  65% {
    clip: rect(35px, 9999px, 23px, 0);
  }
  70% {
    clip: rect(41px, 9999px, 46px, 0);
  }
  75% {
    clip: rect(35px, 9999px, 3px, 0);
  }
  80% {
    clip: rect(41px, 9999px, 96px, 0);
  }
  85% {
    clip: rect(52px, 9999px, 59px, 0);
  }
  90% {
    clip: rect(69px, 9999px, 97px, 0);
  }
  95% {
    clip: rect(10px, 9999px, 71px, 0);
  }
  100% {
    clip: rect(67px, 9999px, 38px, 0);
  }
}
@keyframes glitch-anim2 {
  0% {
    clip: rect(65px, 9999px, 59px, 0);
  }
  5% {
    clip: rect(88px, 9999px, 67px, 0);
  }
  10% {
    clip: rect(94px, 9999px, 7px, 0);
  }
  15% {
    clip: rect(73px, 9999px, 14px, 0);
  }
  20% {
    clip: rect(96px, 9999px, 71px, 0);
  }
  25% {
    clip: rect(13px, 9999px, 35px, 0);
  }
  30% {
    clip: rect(72px, 9999px, 66px, 0);
  }
  35% {
    clip: rect(70px, 9999px, 22px, 0);
  }
  40% {
    clip: rect(13px, 9999px, 98px, 0);
  }
  45% {
    clip: rect(63px, 9999px, 7px, 0);
  }
  50% {
    clip: rect(80px, 9999px, 21px, 0);
  }
  55% {
    clip: rect(27px, 9999px, 52px, 0);
  }
  60% {
    clip: rect(89px, 9999px, 14px, 0);
  }
  65% {
    clip: rect(51px, 9999px, 80px, 0);
  }
  70% {
    clip: rect(2px, 9999px, 37px, 0);
  }
  75% {
    clip: rect(71px, 9999px, 86px, 0);
  }
  80% {
    clip: rect(19px, 9999px, 46px, 0);
  }
  85% {
    clip: rect(82px, 9999px, 8px, 0);
  }
  90% {
    clip: rect(48px, 9999px, 3px, 0);
  }
  95% {
    clip: rect(68px, 9999px, 100px, 0);
  }
  100% {
    clip: rect(47px, 9999px, 2px, 0);
  }
}

/* 列表 */
.ack-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.ack-item {
  margin-bottom: 0.8rem;
  opacity: 0;
  transform: translateY(10px);
  animation: itemIn 0.6s forwards;

  .ack-card {
    position: relative;
    padding: 0.8rem 1rem;
    background: rgba(26, 16, 40, 0.6);
    backdrop-filter: blur(4px);
    border-radius: 12px;
    border: 2px solid transparent;
    background-clip: padding-box;
    transition: all 0.4s;
    overflow: hidden;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: baseline;
    gap: 0.6rem;

    &::before {
      content: "";
      position: absolute;
      inset: -2px;
      background: linear-gradient(
        45deg,
        var(--pink-core),
        var(--blue-glitch),
        var(--purple-mid),
        var(--pink-core)
      );
      border-radius: 14px;
      z-index: -1;
      animation: borderRotate 4s linear infinite;
      opacity: 0;
      transition: opacity 0.3s;
    }
    &:hover::before {
      opacity: 1;
    }
    &:hover {
      transform: translateY(-4px);
      .hologram-scan {
        animation: hologram-scan 1.5s linear infinite;
      }
    }

    .glitch-frame {
      position: absolute;
      inset: 0;
      border: 2px solid transparent;
      border-image: repeating-linear-gradient(
          45deg,
          var(--pink-core),
          var(--blue-glitch),
          var(--purple-mid)
        )
        30;
      pointer-events: none;
      z-index: 2;
      mix-blend-mode: overlay;
      opacity: 0.5;
      animation: glitchFrame 2s infinite;
    }
    @keyframes glitchFrame {
      0%,
      100% {
        border-width: 2px;
        opacity: 0.5;
      }
      25% {
        border-width: 3px;
        opacity: 0.8;
        border-color: var(--blue-glitch);
      }
      50% {
        border-width: 1px;
        opacity: 0.3;
        border-color: var(--pink-core);
      }
      75% {
        border-width: 4px;
        opacity: 0.6;
        border-color: var(--purple-mid);
      }
    }

    .hologram-scan {
      position: absolute;
      left: 0;
      width: 100%;
      height: 3px;
      background: linear-gradient(
        90deg,
        transparent,
        var(--blue-glow),
        var(--pink-core),
        var(--blue-glow),
        transparent
      );
      opacity: 0;
      z-index: 3;
      pointer-events: none;
      filter: blur(2px);
    }

    .nickname {
      font-weight: 700;
      color: var(--pink-core);
      text-shadow: 0 0 8px var(--pink-core);
      white-space: nowrap;
    }

    .suggestion {
      flex: 1;
      color: var(--white-glow);
      font-size: 0.98rem;
      line-height: 1.5;
      word-break: break-word;
    }
  }
}

/* 空状态 */
.empty {
  text-align: center;
  color: var(--blue-glow);
  margin-top: 2rem;
  &::after {
    content: "_";
    animation: blink 1s infinite;
  }
}

/* 页脚 */
.ack-footer {
  text-align: center;
  padding: 1rem 0;
  color: var(--pink-light);
  font-size: 0.9rem;
  background: rgba(26, 16, 40, 0.4);
  backdrop-filter: blur(10px);
  border-top: 1px solid var(--pink-core);
  margin-top: 1.5rem;
  position: relative;
  z-index: 5;
  span::before {
    content: "> ";
    color: var(--blue-glitch);
  }
  span::after {
    content: "_";
    animation: blink 1s infinite;
  }
}

/* 浮动精灵 */
.floating-sprites {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 5;
}
.sprite {
  position: absolute;
  width: 30px;
  height: 30px;
  background: radial-gradient(
    circle at 30% 30%,
    var(--pink-core),
    var(--blue-glitch)
  );
  border-radius: 50%;
  filter: blur(3px) drop-shadow(0 0 8px var(--pink-glow));
  opacity: 0.4;
  pointer-events: auto;
  transition: filter 0.3s;
  &:hover {
    filter: blur(0) drop-shadow(0 0 15px var(--pink-core))
      drop-shadow(0 0 25px var(--blue-glow));
    opacity: 0.8;
  }
}

/* 爱弥斯语录 */
.ai-message {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: rgba(26, 16, 40, 0.9);
  backdrop-filter: blur(10px);
  border: 1px solid var(--pink-core);
  border-radius: 30px;
  padding: 8px 16px;
  color: var(--white-glow);
  font-size: 13px;
  font-family: "Fira Code", monospace;
  z-index: 20;
  animation: cursorAnimation_link 1s infinite step-start;
  box-shadow: 0 0 20px var(--blue-glitch);
  .msg-label {
    color: var(--pink-core);
    margin-right: 8px;
  }
  .msg-content {
    color: var(--blue-glow);
  }
  .msg-cursor {
    animation: blink 1s infinite;
    margin-left: 4px;
  }
}

/* 响应式 */
@media (max-width: 720px) {
  .core-status {
    display: none;
  }
  .ack-container {
    padding: 1.25rem;
  }
  .page-title {
    font-size: 1.8rem;
  }
  .ack-card {
    padding: 0.7rem 0.8rem;
    flex-direction: column;
    align-items: flex-start;
  }
  .nickname {
    font-size: 0.97rem;
  }
  .suggestion {
    font-size: 0.95rem;
  }
  .ai-message {
    bottom: 10px;
    right: 10px;
    font-size: 11px;
    padding: 4px 10px;
  }
}
</style>
