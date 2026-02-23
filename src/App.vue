<template>
  <div id="app">
    <transition name="fade" v-if="showIntro">
      <div class="intro-container" @click="showIntro = false">
        <!-- 背景视频（爱弥斯风格素材，若无可使用静态背景） -->
        <video
          v-if="videoSrc"
          class="video-background"
          :src="videoSrc"
          autoplay
          muted
          loop
          playsinline
        ></video>
        <!-- 无视频时的纯色备选，网格与扫描线通过伪类实现 -->
        <div class="intro-content" aria-live="polite">
          <div class="intro-inner">
            <!-- 打字机显示的爱弥斯语录 -->
            <div class="typewriter-wrap">
              <p class="typewriter glitch" :data-text="displayText">
                {{ displayText }}
              </p>
            </div>
            <!-- 装饰性爱心/电子心 -->
            <span class="heart-icon">❤️</span>
          </div>
        </div>
      </div>
    </transition>
    <div v-else>
      <navbar />
      <main class="main-content">
        <RouterView />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import { RouterView } from "vue-router";
import navbar from "@/components/navbar.vue";

const showIntro = ref(true);
const videoSrc = ref("");

// 爱弥斯风格语录
const lines = [
  "但愿我会让你感到骄傲，但愿我没有让你失望",
  "我是爱弥斯，星炬学院的适格者",
  "即使化为数据，也想守护你的笑容",
  "在数据世界里，为你唱响这首歌",
  "粉发少女的内心，藏着无尽的温柔",
  "电子幽灵，心之所向",
  "星炬的光，永不熄灭",
  "爱弥斯 · 星炬档案，欢迎你的到来",
  "故障中……信号不稳定，但心意不变",
  "愿星炬的光芒，照亮你前行的路",
];

const displayText = ref("");
let typingTimer: number | null = null;

const typingSpeed = 150;

function pickRandomLine(): string {
  const idx = Math.floor(Math.random() * lines.length);
  return lines[idx];
}

function startOnceType(line: string) {
  const reduce =
    window.matchMedia &&
    window.matchMedia("(prefers-reduced-motion: reduce)").matches;
  if (reduce) {
    displayText.value = line;
    return;
  }

  let i = 0;
  typingTimer = window.setInterval(() => {
    i++;
    displayText.value = line.slice(0, i);
    if (i >= line.length) {
      if (typingTimer) {
        clearInterval(typingTimer);
        typingTimer = null;
      }
    }
  }, typingSpeed);
}

onMounted(() => {
  // 根据设备选择视频文件夹（需存放爱弥斯风格视频，若无则留空使用纯背景）
  const isMobile = window.innerWidth <= 768;
  const folder = isMobile ? "mp2" : "mp1";
  const index = Math.floor(Math.random() * 4) + 1; // 1-4
  videoSrc.value = `${folder}/1 (${index}).mp4`; // 示例路径，请按实际命名调整

  // 若视频资源不存在，可静默失败，背景依然美观
  setTimeout(() => {
    showIntro.value = false;
  }, 6000); // 6秒后自动关闭

  const line = pickRandomLine();
  setTimeout(() => startOnceType(line), 200);
});

onBeforeUnmount(() => {
  if (typingTimer) clearInterval(typingTimer);
});
</script>

<style scoped lang="scss">
#app {
  position: relative;
  min-height: 100vh;
  animation: cursorAnimation 1s infinite step-start;
}

/* 淡入淡出动画 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* 爱弥斯风格欢迎页 */
.intro-container {
  --pink-core: #ff8cb0;
  --blue-glitch: #6ed4ff;
  --bg-deep: linear-gradient(145deg, #1a1025 0%, #23182b 100%);
  --grid-color: rgba(255, 140, 176, 0.15);
  --heart-glow: rgba(255, 140, 176, 0.6);

  position: fixed;
  inset: 0;
  background: var(--bg-deep);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;

  // 科技网格
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
    opacity: 0.3;
    mix-blend-mode: overlay;
  }

  // 扫描线动画
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
    animation: scan 8s linear infinite;
  }
}

.video-background {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.25; // 降低透明度以突出网格和文字
  z-index: 0;
  pointer-events: none;
  filter: hue-rotate(-10deg) saturate(1.2); // 增强粉蓝感
}

.intro-content {
  position: relative;
  z-index: 10;
  width: 100%;
  padding: 18px 12px;
  display: flex;
  justify-content: center;
  align-items: center;

  .intro-inner {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 30px;
    background: rgba(0, 0, 0, 0.2);
    backdrop-filter: blur(4px);
    border: 1px solid rgba(255, 140, 176, 0.3);
    border-radius: 60px;
    padding: 20px 40px;
    box-shadow: 0 0 40px var(--heart-glow);

    .typewriter-wrap {
      display: flex;
      align-items: center;

      .typewriter {
        margin: 0;
        font-weight: 800;
        font-size: clamp(32px, 6vw, 56px);
        font-family: "Cinzel", "Noto Serif SC", serif;
        line-height: 1.2;
        background: linear-gradient(
          135deg,
          var(--pink-core) 20%,
          var(--blue-glitch) 80%
        );
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        text-shadow: 2px 2px 0 rgba(110, 212, 255, 0.3),
          -2px -2px 0 var(--pink-core);
        position: relative;

        // Glitch 故障效果
        &.glitch::before,
        &.glitch::after {
          content: attr(data-text);
          position: absolute;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          background: transparent;
          clip: rect(0, 0, 0, 0);
        }
        &.glitch::before {
          left: -2px;
          text-shadow: 2px 0 var(--blue-glitch);
          animation: glitch-before 0.4s infinite linear alternate-reverse;
        }
        &.glitch::after {
          left: 2px;
          text-shadow: -2px 0 var(--pink-core);
          animation: glitch-after 0.3s infinite linear alternate-reverse;
        }
      }
    }

    .heart-icon {
      font-size: 4rem;
      animation: float-heart 3s ease-in-out infinite;
      filter: drop-shadow(0 0 20px var(--heart-glow));
      opacity: 0.9;
    }
  }
}

/* 移动端适配 */
@media (max-width: 768px) {
  .intro-inner {
    flex-direction: column;
    gap: 15px;
    padding: 20px 25px;
    border-radius: 40px;

    .typewriter {
      font-size: clamp(24px, 5vw, 36px);
      text-align: center;
    }

    .heart-icon {
      font-size: 3rem;
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

@keyframes float-heart {
  0% {
    transform: translateY(0) rotate(0deg);
  }
  50% {
    transform: translateY(-12px) rotate(8deg);
  }
  100% {
    transform: translateY(0) rotate(0deg);
  }
}
</style>
