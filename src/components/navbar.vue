<template>
  <header class="app-header">
    <h1 class="title glitch" data-text="爱弥斯 · 星炬档案">
      爱弥斯 · 星炬档案
    </h1>

    <!-- 桌面端右侧操作区（移动端隐藏） -->
    <div class="header-actions desktop-only">
      <div class="musicMp4">
        <video
          v-if="!isPlaying"
          @click="togglePlay"
          src="/小爱4.mp4"
          muted
          loop
          playsinline
        ></video>
        <video
          v-else
          @click="togglePlay"
          src="/小爱4.mp4"
          autoplay
          muted
          loop
          playsinline
        ></video>
      </div>
      <!-- <button
        class="music-btn"
        :class="{ playing: isPlaying }"
        @click="togglePlay"
        aria-label="播放/暂停背景音乐"
      >
        <span class="note-icon">♪</span>
      </button> -->
      <div class="online-count" v-if="onlineCount !== null">
        当前在线：<span class="count">{{ onlineCount }}人</span>
      </div>
    </div>

    <!-- 移动端汉堡按钮 -->
    <button
      class="hamburger"
      @click="toggleMobileNav"
      aria-label="Toggle navigation"
    >
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
    </button>

    <!-- 导航菜单（移动端展开时包含额外控制项） -->
    <nav :class="['nav-links', { 'mobile-open': mobileNavOpen }]">
      <!-- 固定显示的导航项（前5个） -->
      <RouterLink
        v-for="item in mainItems"
        :key="item.name"
        :to="item.path"
        class="nav-item"
        active-class="active-link"
        @click="mobileNavOpen = false"
      >
        {{ item.name }}
      </RouterLink>

      <!-- PC端下拉菜单（更多内部项） -->
      <div class="dropdown pc-only">
        <span class="nav-item dropdown-trigger">
          更多 <span class="arrow" :class="{ open: dropdownOpen }">▼</span>
        </span>
        <div class="dropdown-menu" @mouseleave="dropdownOpen = false">
          <template v-for="item in moreInternalItems" :key="item.name">
            <RouterLink
              :to="item.path"
              class="dropdown-item"
              @click="mobileNavOpen = false"
            >
              {{ item.name }}
            </RouterLink>
          </template>
        </div>
      </div>

      <!-- 霜落映界（独立显示，PC端放在最后） -->
      <a
        href="http://36.150.237.25/#/redirector"
        target="_blank"
        rel="noopener"
        class="nav-item pc-only"
        @click="mobileNavOpen = false"
      >
        霜落映界
      </a>

      <!-- 移动端显示的完整列表（平铺） -->
      <div class="mobile-only full-list">
        <template v-for="item in allMobileItems" :key="item.name">
          <RouterLink
            v-if="!item.external"
            :to="item.path"
            class="nav-item"
            active-class="active-link"
            @click="mobileNavOpen = false"
          >
            {{ item.name }}
          </RouterLink>
          <a
            v-else
            :href="item.path"
            target="_blank"
            rel="noopener"
            class="nav-item"
            @click="mobileNavOpen = false"
          >
            {{ item.name }}
          </a>
        </template>
      </div>

      <!-- 移动端控制区域（仅在菜单展开时显示） -->
      <div class="mobile-controls" v-if="mobileNavOpen">
        <div class="online-count-mobile" v-if="onlineCount !== null">
          <span class="label">心电感应</span>
          <span class="count">{{ onlineCount }}人</span>
        </div>
        <button
          class="music-btn-mobile"
          :class="{ playing: isPlaying }"
          @click="togglePlay"
        >
          <span class="note-icon">♪</span>
          <span>{{ isPlaying ? "暂停音律" : "播放音律" }}</span>
        </button>
      </div>
    </nav>
  </header>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import { io } from "socket.io-client";

// 内部路由项
const internalNavItems = [
  { name: "星炬回响", path: "/", external: false },
  { name: "爱弥斯档案", path: "/timeLine", external: false },
  { name: "心迹留言", path: "/message", external: false },
  { name: "故障影集", path: "/gallery", external: false },
  { name: "心电感应", path: "/talk", external: false },
  { name: "围住阿漂", path: "/game", external: false },
  // { name: "声波频段", path: "/voice", external: false },
  { name: "数据共享", path: "/resources", external: false },
  { name: "音律库", path: "/music", external: false },
  { name: "星炬文库", path: "/wiki", external: false },
  { name: "鸣谢", path: "/thanks", external: false },
];

// 主项（前5个）
const mainItems = internalNavItems.slice(0, 5);

// 更多内部项（第6到第11）
const moreInternalItems = internalNavItems.slice(5);

// 外部链接
const externalItem = {
  name: "霜落映界",
  path: "http://36.150.237.25/#/redirector",
  external: true,
};

// 移动端所有项（内部项+外部项）
const allMobileItems = [...internalNavItems, externalItem];

const mobileNavOpen = ref(false);
function toggleMobileNav() {
  mobileNavOpen.value = !mobileNavOpen.value;
}

// 下拉菜单状态
const dropdownOpen = ref(false);

const siteId = "ams";
const onlineCount = ref<number | null>(null);

const socket: any = io("http://36.150.237.25:3000", {
  query: { siteId },
});

onMounted(() => {
  socket.on("onlineCount", (count: number) => {
    onlineCount.value = count;
  });
});

onBeforeUnmount(() => {
  socket.disconnect();
});

// BGM 控制
const audio = ref<HTMLAudioElement | null>(null);
const isPlaying = ref(false);
const audioError = ref(false);

const togglePlay = () => {
  if (!audio.value) {
    initAudio();
  }
  if (audio.value) {
    if (isPlaying.value) {
      audio.value.pause();
      isPlaying.value = false;
    } else {
      audio.value
        .play()
        .then(() => {
          isPlaying.value = true;
          audioError.value = false;
        })
        .catch((e) => {
          console.error("播放失败:", e);
          audioError.value = true;
          alert("音律加载失败，请稍后重试～");
        });
    }
  }
};

const initAudio = () => {
  if (audio.value) {
    audio.value.pause();
    audio.value = null;
  }
  const songs = [
    "星炬不熄.mp3",
    "碎花.mp3",
    "小小奇迹.mp3",
    "远航星的告别.mp3",
    "靛青宇宙.mp3",
    "那颗星梦见的春日.mp3",
  ];
  const randomSong = songs[Math.floor(Math.random() * songs.length)];
  // console.log('randomSong',randomSong)
  const audioUrl = encodeURI(`http://36.150.237.25:3000/music/${randomSong}`);
  const newAudio = new Audio(audioUrl);
  newAudio.loop = true;
  newAudio.addEventListener("error", (e) => {
    console.error("音频加载错误:", newAudio.error);
    audioError.value = true;
  });
  newAudio.load();
  audio.value = newAudio;
};

onMounted(() => {
  initAudio();

  // 随机故障特效（精细化）
  setInterval(() => {
    const items = document.querySelectorAll(".nav-item:not(.dropdown-trigger)");
    if (items.length && Math.random() > 0.8) {
      const randomItem = items[Math.floor(Math.random() * items.length)];
      randomItem.classList.add("glitch-effect");
      setTimeout(() => randomItem.classList.remove("glitch-effect"), 200);
    }
  }, 4000);
});

onBeforeUnmount(() => {
  if (audio.value) {
    audio.value.pause();
    audio.value = null;
  }
});
</script>

<style scoped lang="scss">
/* 爱弥斯 — 电子幽灵：粉芯 / 故障蓝 / 爱心 / 科技网格（超级精美版） */
.app-header {
  --pink-core: #ff8cb0;
  --pink-light: #ffb3cc;
  --pink-glow: #ffb6d9;
  --blue-glitch: #6ed4ff;
  --blue-glow: #9ae2ff;
  --purple-mid: #e0a0ff; // 过渡色
  --white-glow: #fbefff;
  --bg-deep: linear-gradient(145deg, #1a1028 0%, #281e30 100%);
  --grid-color: rgba(255, 140, 176, 0.15);
  --glitch-shadow: rgba(110, 212, 255, 0.5);
  --heart-glow: rgba(255, 140, 176, 0.7);

  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  height: 70px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 40px;
  background: var(--bg-deep);
  backdrop-filter: blur(12px) saturate(200%);
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.6),
    0 0 0 1px rgba(255, 140, 176, 0.3) inset, 0 0 40px rgba(110, 212, 255, 0.2);
  border-bottom: 1px solid rgba(255, 140, 176, 0.3);

  // 动态彩色光晕层（缓慢移动）
  & > :first-child::before {
    content: "";
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: radial-gradient(
        circle at 30% 50%,
        rgba(255, 140, 176, 0.2) 0%,
        transparent 30%
      ),
      radial-gradient(
        circle at 70% 50%,
        rgba(110, 212, 255, 0.2) 0%,
        transparent 30%
      );
    animation: glow-drift 15s ease-in-out infinite;
    pointer-events: none;
    z-index: -1;
    mix-blend-mode: screen;
  }

  // 科技网格（带微光闪烁）
  &::after {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-image: linear-gradient(
        90deg,
        var(--grid-color) 1px,
        transparent 1px
      ),
      linear-gradient(0deg, var(--grid-color) 1px, transparent 1px);
    background-size: 30px 30px;
    background-position: center center;
    pointer-events: none;
    z-index: 0;
    opacity: 0.2;
    mask: linear-gradient(
      to bottom,
      transparent,
      black 20%,
      black 80%,
      transparent
    );
    animation: grid-flicker 4s infinite;
  }

  // 扫描线（更细腻）
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
      rgba(255, 140, 176, 0.03) 1px,
      transparent 2px,
      transparent 8px
    );
    pointer-events: none;
    z-index: 1;
    animation: scan 12s linear infinite;
    mix-blend-mode: overlay;
  }

  // 底部光带
  &::after {
    content: "";
    position: absolute;
    bottom: -2px;
    left: 0;
    width: 100%;
    height: 4px;
    background: linear-gradient(
      90deg,
      transparent,
      var(--pink-core),
      var(--blue-glitch),
      var(--purple-mid),
      transparent
    );
    filter: blur(4px);
    opacity: 0.6;
    animation: bottom-glow 3s ease-in-out infinite;
  }

  .title {
    position: relative;
    font-family: "Cinzel", "Noto Serif SC", serif;
    font-size: 26px;
    font-weight: 800;
    letter-spacing: 2px;
    color: var(--white-glow);
    text-transform: uppercase;
    z-index: 2;
    background: linear-gradient(
      135deg,
      var(--pink-core) 20%,
      var(--purple-mid) 50%,
      var(--blue-glitch) 80%
    );
    background-clip: text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 0 15px var(--pink-glow), 0 0 30px var(--blue-glow);
    animation: glitch 4s infinite alternate;

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
      left: -2px;
      text-shadow: 2px 0 var(--blue-glitch), -2px 0 var(--pink-core);
      animation: glitch-before 0.3s infinite linear alternate-reverse;
    }

    &::after {
      left: 2px;
      text-shadow: -2px 0 var(--pink-core), 2px 0 var(--blue-glitch);
      animation: glitch-after 0.2s infinite linear alternate-reverse;
    }
  }

  // 桌面端操作区
  .header-actions {
    display: flex;
    align-items: center;
    gap: 20px;
    z-index: 3;
    margin-left: auto;
    margin-right: 20px;

    .music-btn {
      width: 44px;
      height: 44px;
      border-radius: 50%;
      background: radial-gradient(
        circle at 30% 30%,
        var(--pink-light),
        var(--pink-core)
      );
      border: none;
      box-shadow: 0 0 15px var(--heart-glow), 0 4px 10px rgba(0, 0, 0, 0.3),
        0 0 0 2px rgba(255, 255, 255, 0.3) inset;
      animation: cursorAnimation_link 1s infinite step-start;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all 0.3s ease;
      position: relative;
      overflow: hidden;

      .note-icon {
        font-size: 24px;
        color: white;
        text-shadow: 0 0 8px var(--blue-glitch);
        transition: transform 0.3s;
      }

      &::after {
        content: "";
        position: absolute;
        top: -50%;
        left: -50%;
        width: 200%;
        height: 200%;
        background: radial-gradient(
          circle,
          rgba(255, 255, 255, 0.4) 10%,
          transparent 70%
        );
        opacity: 0;
        transition: opacity 0.3s;
      }

      &:hover {
        transform: scale(1.1) rotate(5deg);
        box-shadow: 0 0 30px var(--heart-glow), 0 6px 14px rgba(0, 0, 0, 0.5);
        &::after {
          opacity: 1;
        }
        .note-icon {
          transform: scale(1.2) rotate(10deg);
        }
      }

      &.playing {
        animation: pulse-heart 1.5s infinite;
        .note-icon {
          color: var(--blue-glitch);
        }
      }
    }

    .online-count {
      position: relative;
      padding: 6px 16px;
      font-family: "Noto Sans SC", sans-serif;
      font-size: 1rem;
      color: var(--white-glow);
      background: rgba(0, 0, 0, 0.2);
      border: 1px solid rgba(255, 140, 176, 0.6);
      border-radius: 30px;
      backdrop-filter: blur(8px);
      box-shadow: 0 0 20px rgba(255, 140, 176, 0.3);
      cursor: default;
      overflow: hidden;

      &::before {
        content: "";
        position: absolute;
        top: -50%;
        left: -50%;
        width: 200%;
        height: 200%;
        background: linear-gradient(
          45deg,
          transparent,
          rgba(110, 212, 255, 0.1),
          transparent
        );
        animation: shine 3s infinite;
      }

      .count {
        margin-left: 8px;
        font-weight: 800;
        color: var(--blue-glitch);
        text-shadow: 0 0 8px var(--blue-glitch);
        animation: flicker 2.5s infinite;
      }
    }
  }

  .nav-links {
    display: flex;
    gap: 24px;
    align-items: center;
    z-index: 2;

    .nav-item {
      position: relative;
      color: var(--white-glow);
      font-weight: 600;
      text-decoration: none;
      padding: 6px 4px;
      font-family: "Noto Serif SC", serif;
      font-size: 1.1rem;
      transition: color 0.2s, text-shadow 0.2s;
      overflow: visible;
      white-space: nowrap;

      &::after {
        content: "";
        position: absolute;
        left: 50%;
        bottom: -6px;
        width: 0;
        height: 3px;
        background: linear-gradient(
          90deg,
          var(--pink-core),
          var(--blue-glitch),
          var(--purple-mid),
          var(--pink-core)
        );
        border-radius: 2px;
        transform: translateX(-50%);
        transition: width 0.3s ease;
        box-shadow: 0 0 15px var(--blue-glitch);
        filter: blur(0.8px);
      }

      &::before {
        content: "❤️";
        position: absolute;
        left: 50%;
        top: -20px;
        transform: translateX(-50%) scale(0) rotate(-10deg);
        font-size: 18px;
        color: var(--pink-core);
        text-shadow: 0 0 15px var(--heart-glow);
        transition: transform 0.2s ease, top 0.2s;
        opacity: 0.9;
      }

      &:hover {
        color: var(--pink-light);
        text-shadow: 0 0 8px var(--heart-glow);
        &::after {
          width: 90%;
        }
        &::before {
          transform: translateX(-50%) scale(1) rotate(0deg);
          top: -15px;
        }
      }

      &.active-link {
        color: var(--pink-core);
        font-weight: 700;
        text-shadow: 0 0 10px var(--heart-glow);

        &::after {
          width: 100%;
          background: linear-gradient(
            90deg,
            var(--blue-glitch),
            var(--pink-core),
            var(--blue-glitch)
          );
          box-shadow: 0 0 20px var(--pink-core);
        }

        &::before {
          content: "💖";
          transform: translateX(-50%) scale(1);
          top: -10px;
          animation: float-heart 2s infinite;
        }
      }
    }

    // 下拉菜单（超级精美版）
    .dropdown {
      position: relative;

      .dropdown-trigger {
        animation: cursorAnimation_link 1s infinite step-start;
        display: flex;
        align-items: center;
        gap: 4px;

        .arrow {
          font-size: 0.8rem;
          transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
          &.open {
            transform: rotate(180deg);
          }
        }
      }

      .dropdown-menu {
        position: absolute;
        top: 100%;
        left: 50%;
        transform: translateX(-50%) scale(0.9);
        margin-top: 18px;
        min-width: 160px;
        background: rgba(15, 8, 20, 0.7);
        backdrop-filter: blur(25px);
        border: 1px solid transparent;
        border-image: linear-gradient(
            135deg,
            var(--pink-core),
            var(--blue-glitch)
          )
          1;
        border-radius: 28px;
        padding: 12px 0;
        box-shadow: 0 25px 45px rgba(0, 0, 0, 0.7), 0 0 50px var(--heart-glow);
        opacity: 0;
        visibility: hidden;
        transition: opacity 0.3s, visibility 0.3s,
          transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
        z-index: 100;
        display: flex;
        flex-direction: column;
        gap: 6px;

        // 装饰性粒子（在背景中飘动）
        &::after {
          content: "";
          position: absolute;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          background: radial-gradient(
              circle at 20% 30%,
              rgba(255, 140, 176, 0.1) 0%,
              transparent 30%
            ),
            radial-gradient(
              circle at 80% 70%,
              rgba(110, 212, 255, 0.1) 0%,
              transparent 30%
            );
          pointer-events: none;
          animation: particle-drift 8s ease-in-out infinite;
        }

        &::before {
          content: "";
          position: absolute;
          top: -8px;
          left: 50%;
          transform: translateX(-50%) rotate(45deg);
          width: 16px;
          height: 16px;
          background: rgba(15, 8, 20, 0.7);
          border-left: 1px solid var(--pink-core);
          border-top: 1px solid var(--pink-core);
          border-radius: 4px;
          backdrop-filter: blur(25px);
        }

        .dropdown-item {
          position: relative;
          padding: 10px 24px;
          color: var(--white-glow);
          text-decoration: none;
          font-family: "Noto Serif SC", serif;
          font-size: 1rem;
          text-align: center;
          transition: all 0.25s;
          border-radius: 20px;
          margin: 0 8px;
          z-index: 2;

          // 悬浮时的爱心光晕
          &::after {
            content: "❤️";
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, -50%) scale(0);
            font-size: 2rem;
            color: var(--pink-core);
            opacity: 0;
            transition: transform 0.3s ease, opacity 0.3s;
            z-index: -1;
            filter: blur(4px);
          }

          &:hover {
            background: rgba(0, 0, 0, 0.3);
            color: var(--pink-light);
            text-shadow: 0 0 15px var(--heart-glow);
            transform: scale(1.03) translateY(-2px);

            &::after {
              transform: translate(-50%, -50%) scale(1.5);
              opacity: 0.4;
            }
          }

          &.active-link {
            background: rgba(255, 140, 176, 0.2);
            color: var(--pink-core);
            font-weight: 600;
            box-shadow: inset 0 0 15px rgba(255, 140, 176, 0.3);
          }
        }
      }

      &:hover .dropdown-menu {
        opacity: 1;
        visibility: visible;
        transform: translateX(-50%) scale(1);
      }
    }

    // 移动端完整列表（默认隐藏）
    .mobile-only.full-list {
      display: none;
    }

    // 移动端控制区域（默认隐藏）
    .mobile-controls {
      display: none;
    }
  }

  .hamburger {
    display: none;
    flex-direction: column;
    justify-content: space-around;
    width: 32px;
    height: 26px;
    background: none;
    border: none;
    animation: cursorAnimation_link 1s infinite step-start;
    padding: 0;
    z-index: 5;

    span {
      display: block;
      width: 100%;
      height: 3px;
      background: linear-gradient(90deg, var(--pink-core), var(--blue-glitch));
      border-radius: 3px;
      transition: transform 0.3s cubic-bezier(0.68, -0.55, 0.27, 1.55),
        opacity 0.2s;
      box-shadow: 0 0 8px var(--heart-glow);
    }

    span.open:nth-child(1) {
      transform: translateY(11px) rotate(45deg);
      background: linear-gradient(90deg, var(--blue-glitch), var(--pink-core));
    }
    span.open:nth-child(2) {
      opacity: 0;
      transform: scaleX(0);
    }
    span.open:nth-child(3) {
      transform: translateY(-11px) rotate(-45deg);
      background: linear-gradient(90deg, var(--pink-core), var(--blue-glitch));
    }
  }

  // 响应式工具类
  .pc-only {
    display: flex;
  }
  .mobile-only {
    display: none;
  }

  // 移动端样式
  @media (max-width: 768px) {
    padding: 0 20px;

    .title {
      font-size: 20px;
      display: block;
    }

    .desktop-only {
      display: none;
    }

    .hamburger {
      display: flex;
    }

    .pc-only {
      display: none !important;
    }

    .mobile-only {
      display: block;
    }

    .nav-links {
      position: absolute;
      top: 70px;
      left: 0;
      right: 0;
      flex-direction: column;
      background: rgba(15, 8, 20, 0.95);
      backdrop-filter: blur(25px);
      gap: 0;
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.5s cubic-bezier(0.4, 0, 0.2, 1);
      border-top: 2px solid var(--pink-core);
      box-shadow: 0 30px 40px rgba(0, 0, 0, 0.7),
        inset 0 0 30px rgba(255, 140, 176, 0.1);

      &.mobile-open {
        max-height: 100vh;
      }

      > .nav-item {
        display: none;
      }

      .mobile-only.full-list {
        display: flex;
        flex-direction: column;
        width: 100%;
        animation: slideIn 0.3s ease-out;

        .nav-item {
          display: block;
          width: 100%;
          padding: 16px 24px;
          border-bottom: 1px solid rgba(255, 140, 176, 0.15);
          text-align: center;
          font-size: 1.2rem;
          transition: background 0.2s, padding 0.2s;

          &::before,
          &::after {
            display: none;
          }

          &:hover {
            background: rgba(255, 140, 176, 0.1);
            padding-left: 30px;
            padding-right: 30px;
          }

          &.active-link {
            background: linear-gradient(
              90deg,
              rgba(255, 140, 176, 0.2),
              transparent
            );
            border-left: 4px solid var(--pink-core);
          }
        }
      }

      // 移动端控制区域
      .mobile-controls {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 16px;
        padding: 24px;
        width: 100%;
        border-top: 1px solid rgba(255, 140, 176, 0.3);
        margin-top: 10px;
        background: rgba(0, 0, 0, 0.2);

        .online-count-mobile {
          font-family: "Noto Sans SC", sans-serif;
          font-size: 1.2rem;
          color: var(--white-glow);
          background: rgba(0, 0, 0, 0.3);
          border: 1px solid rgba(255, 140, 176, 0.6);
          border-radius: 40px;
          padding: 10px 28px;
          width: fit-content;
          backdrop-filter: blur(8px);
          box-shadow: 0 0 20px rgba(255, 140, 176, 0.4);

          .label {
            margin-right: 10px;
            opacity: 0.9;
          }

          .count {
            font-weight: 800;
            color: var(--blue-glitch);
            text-shadow: 0 0 10px var(--blue-glitch);
            animation: flicker 2.5s infinite;
          }
        }

        .music-btn-mobile {
          display: flex;
          align-items: center;
          justify-content: center;
          gap: 10px;
          background: radial-gradient(
            circle at 30% 30%,
            var(--pink-light),
            var(--pink-core)
          );
          border: none;
          border-radius: 50px;
          padding: 12px 32px;
          color: white;
          font-size: 1.2rem;
          font-weight: 600;
          box-shadow: 0 0 20px var(--heart-glow), 0 6px 14px rgba(0, 0, 0, 0.5);
          animation: cursorAnimation_link 1s infinite step-start;
          transition: all 0.2s;

          .note-icon {
            font-size: 1.6rem;
            text-shadow: 0 0 8px var(--blue-glitch);
          }

          &.playing {
            animation: pulse-heart 1.5s infinite;
            .note-icon {
              color: var(--blue-glitch);
            }
          }

          &:hover {
            transform: scale(1.05);
            box-shadow: 0 0 30px var(--heart-glow);
          }
        }
      }
    }
  }
}
.musicMp4 {
  video {
    height: 60px;
    border-radius: 50%;
    width: 60px;
  }
}
/* 动画定义（超级精美版） */
@keyframes glow-drift {
  0% {
    transform: translate(0, 0) rotate(0deg);
    opacity: 0.5;
  }
  50% {
    transform: translate(-2%, -1%) rotate(2deg);
    opacity: 0.8;
  }
  100% {
    transform: translate(0, 0) rotate(0deg);
    opacity: 0.5;
  }
}

@keyframes grid-flicker {
  0% {
    opacity: 0.15;
  }
  50% {
    opacity: 0.25;
  }
  100% {
    opacity: 0.15;
  }
}

@keyframes bottom-glow {
  0% {
    opacity: 0.4;
    filter: blur(4px);
  }
  50% {
    opacity: 0.8;
    filter: blur(6px);
  }
  100% {
    opacity: 0.4;
    filter: blur(4px);
  }
}

@keyframes particle-drift {
  0% {
    transform: translate(0, 0);
  }
  50% {
    transform: translate(2%, 1%);
  }
  100% {
    transform: translate(0, 0);
  }
}

@keyframes scan {
  0% {
    background-position: 0 0;
  }
  100% {
    background-position: 0 100%;
  }
}

@keyframes shine {
  0% {
    transform: translateX(-100%) rotate(45deg);
  }
  20% {
    transform: translateX(100%) rotate(45deg);
  }
  100% {
    transform: translateX(100%) rotate(45deg);
  }
}

@keyframes glitch {
  0% {
    text-shadow: 2px 2px 0 var(--glitch-shadow), -2px -2px 0 var(--pink-core),
      0 0 15px var(--blue-glow);
  }
  25% {
    text-shadow: -3px 1px 0 var(--blue-glitch), 3px -1px 0 var(--pink-core),
      0 0 20px var(--purple-mid);
  }
  50% {
    text-shadow: 2px -2px 0 var(--blue-glitch), -2px 2px 0 var(--pink-core),
      0 0 25px var(--pink-glow);
  }
  75% {
    text-shadow: 3px 2px 0 var(--pink-core), -3px -2px 0 var(--blue-glitch),
      0 0 15px var(--blue-glow);
  }
  100% {
    text-shadow: -2px 3px 0 var(--pink-core), 2px -3px 0 var(--blue-glitch),
      0 0 20px var(--purple-mid);
  }
}

@keyframes glitch-before {
  0% {
    clip: rect(44px, 9999px, 56px, 0);
    transform: skew(0.5deg);
  }
  50% {
    clip: rect(12px, 9999px, 24px, 0);
    transform: skew(-0.5deg);
  }
  100% {
    clip: rect(86px, 9999px, 98px, 0);
    transform: skew(0.2deg);
  }
}

@keyframes glitch-after {
  0% {
    clip: rect(86px, 9999px, 98px, 0);
    transform: skew(-0.3deg);
  }
  50% {
    clip: rect(38px, 9999px, 52px, 0);
    transform: skew(0.3deg);
  }
  100% {
    clip: rect(14px, 9999px, 28px, 0);
    transform: skew(-0.1deg);
  }
}

@keyframes pulse-heart {
  0% {
    box-shadow: 0 0 15px var(--heart-glow), 0 4px 10px rgba(0, 0, 0, 0.3);
  }
  50% {
    box-shadow: 0 0 40px var(--heart-glow), 0 6px 14px rgba(0, 0, 0, 0.5);
  }
  100% {
    box-shadow: 0 0 15px var(--heart-glow), 0 4px 10px rgba(0, 0, 0, 0.3);
  }
}

@keyframes flicker {
  0%,
  100% {
    opacity: 1;
    text-shadow: 0 0 8px var(--blue-glitch), 0 0 15px var(--pink-core);
  }
  30% {
    opacity: 0.7;
    text-shadow: 0 0 15px var(--pink-core), 0 0 25px var(--blue-glitch);
  }
  70% {
    opacity: 1;
    text-shadow: 0 0 8px var(--blue-glitch);
  }
}

@keyframes float-heart {
  0% {
    transform: translateX(-50%) scale(1) translateY(0);
    filter: drop-shadow(0 0 8px var(--heart-glow));
  }
  50% {
    transform: translateX(-50%) scale(1.2) translateY(-6px);
    filter: drop-shadow(0 0 20px var(--heart-glow));
  }
  100% {
    transform: translateX(-50%) scale(1) translateY(0);
    filter: drop-shadow(0 0 8px var(--heart-glow));
  }
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.nav-item.glitch-effect {
  animation: glitch-text 0.2s 2;
}
@keyframes glitch-text {
  0% {
    transform: skew(0deg);
    opacity: 1;
    filter: blur(0);
  }
  20% {
    transform: skew(8deg) translateX(3px);
    opacity: 0.7;
    color: var(--blue-glitch);
    filter: blur(0.5px);
  }
  40% {
    transform: skew(-8deg) translateX(-3px);
    opacity: 0.7;
    color: var(--pink-core);
    filter: blur(0.5px);
  }
  60% {
    transform: skew(4deg);
    opacity: 0.9;
  }
  100% {
    transform: skew(0deg);
    opacity: 1;
    filter: blur(0);
  }
}
</style>
