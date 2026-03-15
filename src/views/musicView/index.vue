<template>
  <section
    class="cantarella-player"
    @keydown.space.prevent="onSpace"
    tabindex="0"
    ref="rootEl"
    aria-label="爱弥斯 音乐播放器"
  >
    <!-- 故障光效背景层 + 数据粒子 + 扫描线 -->
    <div class="glitch-bg"></div>
    <div class="data-particles"></div>
    <div class="scan-line"></div>

    <!-- 核心能量状态 -->
    <div class="core-status">
      <span class="pulse-dot"></span>
      <span class="status-text">AI-EMS CORE: 98%</span>
    </div>

    <!-- 主播放器区域（新垂直布局） -->
    <div class="player-main">
      <!-- 顶部：封面区（包含视频） -->
      <div class="cover-section">
        <div class="cover" :style="coverStyle">
          <video
            v-if="videoSrc"
            class="video-background"
            :src="videoSrc"
            autoplay
            muted
            loop
            playsinline
            aria-hidden="true"
            tabindex="-1"
            :class="videoClass"
          ></video>
          <!-- 加载遮罩 -->
          <div v-if="loadingAudio" class="loading-overlay" aria-hidden="true">
            <div class="spinner" />
            <div class="loading-text">加载中…</div>
          </div>
        </div>
      </div>

      <!-- 中部：控制区 -->
      <div class="controls-section">
        <div class="controls">
          <div class="title" :title="current?.title || '未选择曲目'">
            {{ current?.title || "未选择曲目" }}
          </div>
          <div class="meta">
            <span class="time">{{ formatTime(currentTime) }}</span>
            <span class="divider">/</span>
            <span class="time">{{ formatTime(duration) }}</span>
          </div>

          <!-- 进度条 -->
          <div
            class="progress-wrap"
            ref="progressWrap"
            @click="seekByClick"
            @pointerdown.prevent="onPointerDownProgress"
            role="slider"
            :aria-valuemin="0"
            :aria-valuemax="duration"
            :aria-valuenow="currentTime"
            aria-label="进度条"
          >
            <div class="progress-bar">
              <div
                class="progress"
                :style="{ width: progressPercent + '%' }"
              ></div>
            </div>
            <div
              class="progress-handle"
              :style="{ left: progressPercent + '%' }"
              aria-hidden="true"
            ></div>
          </div>

          <!-- 控件行 -->
          <div class="btns">
            <button class="icon" @click="prev" aria-label="上一首">⟵</button>
            <button
              class="play"
              @click="togglePlay"
              :aria-pressed="playing"
              :aria-label="playing ? '暂停' : '播放'"
            >
              <span v-if="!playing">▶</span>
              <span v-else>▌▌</span>
            </button>
            <button class="icon" @click="next" aria-label="下一首">⟶</button>

            <div class="modes" role="group" aria-label="播放模式">
              <button
                :class="{ active: shuffle }"
                @click="toggleShuffle"
                title="随机播放"
              >
                🔀
              </button>
              <button
                :class="{ active: repeatMode !== 'off' }"
                @click="toggleRepeat"
                title="循环模式"
              >
                🔁
              </button>
            </div>

            <div class="volume" aria-label="音量控制">
              <input
                type="range"
                min="0"
                max="1"
                step="0.01"
                v-model.number="volume"
                aria-label="音量"
              />
            </div>
          </div>

          <div v-if="errorMessage" class="error-msg" role="status">
            {{ errorMessage }}
          </div>
        </div>
      </div>

      <!-- 底部：播放列表（可折叠） -->
      <div
        class="list-section"
        :class="{ collapsed: !playlistOpen && isMobile }"
        role="region"
        aria-label="播放列表"
      >
        <div class="playlist-header">
          <div class="left-head">
            <h3>播放列表</h3>

            <div class="api-hint">
              {{ loading ? "加载中…" : list.length ? "" : "目录为空" }}
            </div>
          </div>

          <div class="search-wrap">
            <input
              v-model="searchTerm"
              @input="onSearchInput"
              placeholder="搜索曲名..."
              aria-label="搜索曲目"
            />
            <button
              v-if="searchTerm"
              class="clear"
              @click="clearSearch"
              aria-label="清除搜索"
            >
              ✕
            </button>
          </div>
        </div>

        <div class="list-area">
          <div v-if="loading" class="list-loading">
            <div class="small-spinner" />
            加载目录...
          </div>

          <ul class="playlist" role="list">
            <li
              v-for="(item, idx) in filteredList"
              :key="item.name || idx"
              :class="{ active: current && item.name === current.name }"
              @click="selectTrack(idx)"
              tabindex="0"
              @keyup.enter="selectTrack(idx)"
              role="listitem"
              :aria-current="idx === index ? 'true' : 'false'"
            >
              <div class="left-col">
                <div class="dot" aria-hidden="true"></div>
                <div class="title" :title="item.title">{{ item.title }}</div>
              </div>
              <div class="right-col">
                <div class="len">
                  {{ item.duration ? formatTime(item.duration) : "--:--" }}
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- audio 元素 -->
    <audio
      ref="audioRef"
      @timeupdate="onTimeUpdate"
      @ended="onEnded"
      @loadedmetadata="onLoadedMetadata"
      @error="onAudioError"
      preload="metadata"
    ></audio>

    <!-- 浮动精灵（少量） -->
    <div class="floating-sprites">
      <div
        v-for="(sprite, i) in spriteList"
        :key="i"
        class="sprite"
        :style="{ top: sprite.top + 'px', left: sprite.left + 'px' }"
      ></div>
    </div>

    <!-- 爱弥斯核心语录（可选） -->
    <div class="ai-message" v-if="showAiMessage" @click="nextMessage">
      <span class="msg-label">爱弥斯说:</span>
      <span class="msg-content">{{ currentMessage }}</span>
      <span class="msg-cursor">_</span>
    </div>
  </section>
</template>

<script setup lang="ts">
// 脚本部分完全保持不变（功能接口不变）
import {
  ref,
  computed,
  onMounted,
  onBeforeUnmount,
  watch,
  nextTick,
} from "vue";
import { getMusicList, getMusicUrl } from "@/api/modules/music";
import { gsap } from "gsap";

type MusicItem = {
  name: string;
  title: string;
  url?: string;
  duration?: number | null;
};

const list = ref<MusicItem[]>([]);
const loading = ref(false);
const index = ref<number>(-1);
const playing = ref(false);
const audioRef = ref<HTMLAudioElement | null>(null);
const currentTime = ref<number>(0);
const duration = ref<number>(0);
const volume = ref<number>(
  Number(localStorage.getItem("cantarella_volume") ?? 0.8)
);
const shuffle = ref<boolean>(false);
const repeatMode = ref<"off" | "one" | "all">("off");

const rootEl = ref<HTMLElement | null>(null);
const progressWrap = ref<HTMLElement | null>(null);
const dragging = ref(false);
const playlistOpen = ref(true);
const errorMessage = ref<string | null>(null);
const loadingAudio = ref(false);

// 根据窗口宽度判断移动端
const isMobile = ref<boolean>(window.innerWidth <= 920);
window.addEventListener("resize", () => {
  isMobile.value = window.innerWidth <= 920;
});

// 随机选择视频源（mp1/mp2 里）
const videoSrc = ref("");
const videoClass = ref("");

// 搜索相关
const searchTerm = ref("");
let searchTimer: any = null;
const searchDebounceMs = 240;

// 当前曲目与进度计算
const current = computed(() =>
  index.value >= 0 && list.value[index.value] ? list.value[index.value] : null
);
const progressPercent = computed(() =>
  duration.value
    ? Math.min(100, Math.max(0, (currentTime.value / duration.value) * 100))
    : 0
);

// 封面渐变（爱弥斯风格）
const coverStyle = computed(() => {
  const t = current.value?.title || "cantarella";
  let hash = 0;
  for (let i = 0; i < t.length; i++)
    hash = (hash << 5) - hash + t.charCodeAt(i);
  const r1 = (Math.abs(hash) % 120) + 40;
  const r2 = (Math.abs(hash * 3) % 120) + 40;
  return {
    background: `radial-gradient(circle at 28% 28%, rgba(255,140,176,0.1), transparent 12%), linear-gradient(135deg, rgba(${r1},${r2},255,0.15), rgba(110,212,255,0.1))`,
  };
});

// 过滤后的列表
const filteredList = computed(() => {
  const term = (searchTerm.value || "").trim().toLowerCase();
  if (!term) return list.value;
  return list.value.filter((i) => (i.title || "").toLowerCase().includes(term));
});

// 获取列表
async function fetchList() {
  loading.value = true;
  try {
    const res = await getMusicList();
    const items =
      res?.ok && Array.isArray(res.list)
        ? res.list
        : Array.isArray(res)
        ? res
        : res?.list ?? [];
    list.value = items.map((it: any) => ({
      name: it.name,
      title: it.title ?? (it.name ? it.name.replace(/\.mp3$/i, "") : "未知"),
      url: getMusicUrl(it.name),
      duration: null,
    }));
  } catch (e) {
    console.error("获取音乐列表失败", e);
    list.value = [];
    errorMessage.value = "加载目录失败";
  } finally {
    loading.value = false;
  }
}

// 设置并预检音源
async function safeSetSrc(url: string) {
  const a = audioRef.value!;
  errorMessage.value = null;
  loadingAudio.value = true;
  try {
    try {
      const head = await fetch(url, { method: "HEAD" });
      if (!head.ok) throw new Error(`资源响应 ${head.status}`);
      const ct = head.headers.get("content-type") || "";
      if (!ct.includes("audio")) {
        console.warn("content-type 不是 audio:", ct);
      }
    } catch (e) {
      // 忽略 HEAD 失败
    }
    a.src = url;
    a.load();
  } catch (err) {
    console.error("设置音源失败", err);
    errorMessage.value = "无法加载音频资源";
    throw err;
  }
}

async function loadCurrent(doPlay = false) {
  const a = audioRef.value;
  const curr = current.value;
  if (!a || !curr) return;
  a.pause();
  duration.value = 0;
  currentTime.value = 0;
  try {
    await safeSetSrc(curr.url || getMusicUrl(curr.name));
    if (doPlay) await play();
  } catch {
    playing.value = false;
    loadingAudio.value = false;
  }
}

async function play() {
  const a = audioRef.value;
  if (!a) return;
  try {
    await a.play();
    playing.value = true;
    errorMessage.value = null;
  } catch (e: any) {
    console.warn("播放失败", e);
    playing.value = false;
    errorMessage.value = "播放被浏览器阻止或资源不可用";
  }
}
function pause() {
  audioRef.value?.pause();
  playing.value = false;
}
function togglePlay() {
  if (!audioRef.value) return;
  if (playing.value) pause();
  else play();
}

function selectTrack(idxInFiltered: number) {
  const item = filteredList.value[idxInFiltered];
  if (!item) return;
  // 用唯一字段（如 name）在原始列表中找到正确索引
  const originalIndex = list.value.findIndex((it) => it.name === item.name);
  if (originalIndex === -1) return;
  index.value = originalIndex;
  loadCurrent(true);
}
// 音频事件
function onTimeUpdate(e: Event) {
  const t = e.target as HTMLAudioElement;
  currentTime.value = t.currentTime || 0;
}
function onLoadedMetadata(e: Event) {
  const t = e.target as HTMLAudioElement;
  duration.value = isFinite(t.duration) ? t.duration : 0;
  if (current.value && !current.value.duration)
    current.value.duration = duration.value;
  loadingAudio.value = false;
}
function onEnded() {
  loadingAudio.value = false;
  if (repeatMode.value === "one") {
    if (audioRef.value) {
      audioRef.value.currentTime = 0;
      play();
    }
    return;
  }
  if (shuffle.value) {
    playRandom();
    return;
  }
  if (index.value < list.value.length - 1) selectTrack(index.value + 1);
  else {
    if (repeatMode.value === "all") selectTrack(0);
    else playing.value = false;
  }
}
function onAudioError(e: Event) {
  const a = audioRef.value;
  console.error("audio error", a?.error);
  errorMessage.value = "音频播放出错";
  playing.value = false;
  loadingAudio.value = false;
}

// 上一/下一/随机
function next() {
  if (!list.value.length) return;
  if (shuffle.value) {
    playRandom();
    return;
  }
  if (index.value < list.value.length - 1) selectTrack(index.value + 1);
  else if (repeatMode.value === "all") selectTrack(0);
}
function prev() {
  if (!audioRef.value) return;
  if (audioRef.value.currentTime > 4) {
    audioRef.value.currentTime = 0;
    return;
  }
  if (index.value > 0) selectTrack(index.value - 1);
  else if (repeatMode.value === "all") selectTrack(list.value.length - 1);
}
function playRandom() {
  if (!list.value.length) return;
  if (list.value.length === 1) {
    selectTrack(0);
    return;
  }
  let i = index.value;
  while (i === index.value) i = Math.floor(Math.random() * list.value.length);
  selectTrack(i);
}

// 进度条：点击 & 拖拽
function seekByClick(e: MouseEvent | TouchEvent) {
  if (!progressWrap.value || !duration.value || !audioRef.value) return;
  const rect = progressWrap.value.getBoundingClientRect();
  const clientX =
    (e as MouseEvent).clientX ?? (e as TouchEvent).touches?.[0]?.clientX;
  if (clientX == null) return;
  const x = Math.min(Math.max(0, clientX - rect.left), rect.width);
  const ratio = x / rect.width;
  audioRef.value.currentTime = ratio * duration.value;
  currentTime.value = audioRef.value.currentTime;
}
function onPointerDownProgress(e: PointerEvent) {
  if (!progressWrap.value || !audioRef.value || !duration.value) return;
  dragging.value = true;
  (e.target as Element).setPointerCapture?.(e.pointerId);
  window.addEventListener("pointermove", onPointerMoveProgress);
  window.addEventListener("pointerup", onPointerUpProgress);
  handlePointer(e);
}
function onPointerMoveProgress(e: PointerEvent) {
  handlePointer(e);
}
function onPointerUpProgress(e: PointerEvent) {
  dragging.value = false;
  window.removeEventListener("pointermove", onPointerMoveProgress);
  window.removeEventListener("pointerup", onPointerUpProgress);
}
function handlePointer(e: PointerEvent) {
  if (!progressWrap.value || !audioRef.value || !duration.value) return;
  const rect = progressWrap.value.getBoundingClientRect();
  const x = Math.min(Math.max(0, e.clientX - rect.left), rect.width);
  const ratio = x / rect.width;
  audioRef.value.currentTime = ratio * duration.value;
  currentTime.value = audioRef.value.currentTime;
}

// 音量持久化
watch(volume, (v) => {
  if (audioRef.value) audioRef.value.volume = v;
  localStorage.setItem("cantarella_volume", String(v));
});

// 模式切换
function toggleShuffle() {
  shuffle.value = !shuffle.value;
}
function toggleRepeat() {
  if (repeatMode.value === "off") repeatMode.value = "all";
  else if (repeatMode.value === "all") repeatMode.value = "one";
  else repeatMode.value = "off";
}

// 键盘空格
function onSpace() {
  if (
    document.activeElement &&
    ["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)
  )
    return;
  togglePlay();
}

// 搜索防抖
function onSearchInput() {
  if (searchTimer) clearTimeout(searchTimer);
  searchTimer = setTimeout(() => {
    searchTimer = null;
  }, searchDebounceMs);
}
function clearSearch() {
  searchTerm.value = "";
}

// 格式化时间
function formatTime(sec?: number) {
  if (!sec || !isFinite(sec)) return "--:--";
  const s = Math.floor(sec % 60);
  const m = Math.floor(sec / 60);
  return `${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`;
}

// 爱弥斯语录（简单版）
const messages = [
  "音乐数据流稳定...",
  "这首歌的量子态很美妙。",
  "要循环播放吗？",
  "故障率低于阈值。",
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

// 生命周期
onMounted(async () => {
  audioRef.value =
    (document.querySelector(".cantarella-player audio") as HTMLAudioElement) ??
    null;
  if (audioRef.value) audioRef.value.volume = volume.value;

  const isM = isMobile.value;
  const folder = "/mp1";
  const idx = Math.floor(Math.random() * 4) + 1;
  videoSrc.value = `${folder}/1 (${idx}).mp4`;
  videoClass.value = "landscape";

  await fetchList();

  window.addEventListener("keydown", globalKeydown);

  // 初始化语录轮换
  msgInterval = window.setInterval(() => {
    nextMessage();
  }, 8000);

  // 初始化浮动精灵
  const spriteCount = 4;
  const spriteSize = 40;
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
        { opacity: 0.5, scale: 1, duration: 1, delay: idx * 0.1 }
      );
      gsap.to(el, {
        y: "+=30",
        x: "+=15",
        duration: 4 + Math.random() * 2,
        yoyo: true,
        repeat: -1,
        ease: "sine.inOut",
        modifiers: {
          x: (x) => {
            let val = parseFloat(x);
            const maxX = window.innerWidth - 40;
            if (val < 0) val = 0;
            if (val > maxX) val = maxX;
            return val + "px";
          },
          y: (y) => {
            let val = parseFloat(y);
            const maxY = window.innerHeight - 40;
            if (val < 0) val = 0;
            if (val > maxY) val = maxY;
            return val + "px";
          },
        },
      });
      el.addEventListener("mouseenter", () => {
        gsap.killTweensOf(el);
        gsap.to(el, {
          x: "+=" + (Math.random() - 0.5) * 200,
          y: "+=" + (Math.random() - 0.5) * 200,
          duration: 0.8,
          ease: "back.out(2)",
        });
      });
    });
  });
});

onBeforeUnmount(() => {
  window.removeEventListener("keydown", globalKeydown);
  if (msgInterval) clearInterval(msgInterval);
});

// 全局键盘
function globalKeydown(e: KeyboardEvent) {
  if (e.code === "Space") {
    if (
      document.activeElement &&
      ["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)
    )
      return;
    e.preventDefault();
    togglePlay();
  } else if (e.code === "Escape") {
    pause();
  }
}
</script>

<style scoped lang="scss">
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

/* ===== 全局样式 ===== */
.cantarella-player {
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
  padding: 20px;
  min-height: 100vh;
  background: var(--bg-deep);
  color: var(--white-glow);
  font-family: "JetBrains Mono", "Fira Code", "PingFang SC", monospace;
  position: relative;
  overflow-x: hidden;
  outline: none;
  display: flex;
  flex-direction: column;
  align-items: center;
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

/* 主播放器区域（垂直布局） */
.player-main {
  width: 100%;
  max-width: 900px;
  margin: 0 auto;
  position: relative;
  z-index: 5;
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding-top: 60px;
}

/* 封面区 */
.cover-section {
  width: 100%;
  display: flex;
  justify-content: center;
}

.cover {
  width: 100%;
  max-width: 500px;
  height: 280px;
  border-radius: 16px;
  background: var(--bg-deep);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  position: relative;
  box-shadow: 0 0 30px var(--blue-glitch),
    inset 0 0 20px rgba(255, 140, 176, 0.1);
  border: 2px solid transparent;
  background-clip: padding-box;

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
    border-radius: 18px;
    z-index: -1;
    animation: borderRotate 4s linear infinite;
    opacity: 0.6;
  }
}

.video-background {
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.9;
  filter: saturate(0.92) contrast(0.95) blur(0.2px);
}
.video-background.landscape {
  aspect-ratio: 16 / 9;
}
.video-background.portrait {
  aspect-ratio: 9 / 16;
  width: auto;
  height: 110%;
}

/* 加载遮罩 */
.loading-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(26, 16, 40, 0.7);
  backdrop-filter: blur(5px);
  z-index: 8;
}
.spinner,
.small-spinner {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: 4px solid rgba(255, 255, 255, 0.1);
  border-top-color: var(--pink-core);
  animation: spin 1s linear infinite;
}
.small-spinner {
  width: 18px;
  height: 18px;
  border-width: 3px;
  border-top-color: var(--blue-glitch);
  margin-right: 8px;
}
.loading-text {
  margin-top: 8px;
  color: var(--white-glow);
  font-weight: 700;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* 控制区 */
.controls-section {
  width: 100%;
  background: rgba(26, 16, 40, 0.6);
  backdrop-filter: blur(10px);
  border: 1px solid var(--pink-core);
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 0 30px var(--blue-glitch),
    inset 0 0 20px rgba(255, 140, 176, 0.1);
}

.controls {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.title {
  font-size: 1.2rem;
  font-weight: 900;
  color: var(--pink-core);
  letter-spacing: 0.5px;
  text-shadow: 0 0 8px var(--pink-core);
}

.meta {
  font-size: 0.95rem;
  color: var(--blue-glow);
  display: flex;
  gap: 8px;
  align-items: center;
}

/* 进度条 */
.progress-wrap {
  animation: cursorAnimation_link 1s infinite step-start;
  touch-action: none;
  margin-top: 8px;
}
.progress-bar {
  height: 8px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 10px;
  overflow: hidden;
  position: relative;
  border: 1px solid rgba(255, 140, 176, 0.2);
}
.progress {
  height: 100%;
  width: 0%;
  background: linear-gradient(90deg, var(--pink-core), var(--blue-glitch));
  box-shadow: 0 0 15px var(--pink-core);
  transition: width 120ms linear;
}
.progress-handle {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: var(--white-glow);
  transform: translateX(-50%);
  position: relative;
  top: -3px;
  box-shadow: 0 0 20px var(--blue-glitch);
  border: 2px solid var(--pink-core);
}

/* 按钮行 */
.btns {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
  margin-top: 8px;
}
.icon,
.play {
  border: none;
  background: transparent;
  color: var(--white-glow);
  font-size: 20px;
  animation: cursorAnimation_link 1s infinite step-start;
  padding: 8px;
  border-radius: 8px;
  transition: all 0.3s;
  &:hover {
    transform: translateY(-3px);
    text-shadow: 0 0 15px var(--pink-core), 0 0 30px var(--blue-glitch);
  }
}
.play {
  font-size: 22px;
  background: rgba(255, 140, 176, 0.1);
  border: 1px solid var(--pink-core);
  padding: 8px 16px;
  border-radius: 20px;
  box-shadow: 0 0 15px var(--heart-glow);
  &:hover {
    background: var(--pink-core);
    color: #1a1028;
    box-shadow: 0 0 30px var(--pink-core), 0 0 60px var(--blue-glitch);
  }
}

.modes button {
  margin-right: 6px;
  background: transparent;
  border: 1px solid rgba(255, 140, 176, 0.3);
  padding: 6px 10px;
  border-radius: 8px;
  animation: cursorAnimation_link 1s infinite step-start;
  color: var(--white-glow);
  transition: all 0.3s;
  &.active {
    background: rgba(255, 140, 176, 0.2);
    border-color: var(--pink-core);
    box-shadow: 0 0 15px var(--pink-core);
  }
  &:hover {
    border-color: var(--blue-glitch);
  }
}

.volume input[type="range"] {
  width: 100px;
  background: transparent;
  accent-color: var(--pink-core);
}

.error-msg {
  color: #ff6b85;
  font-weight: 700;
  font-size: 0.9rem;
  text-shadow: 0 0 8px #ff6b85;
}

/* 列表区 */
.list-section {
  width: 100%;
  background: rgba(26, 16, 40, 0.6);
  backdrop-filter: blur(10px);
  border: 1px solid var(--pink-core);
  border-radius: 16px;
  padding: 16px;
  box-shadow: 0 0 30px var(--blue-glitch),
    inset 0 0 20px rgba(255, 140, 176, 0.1);
  transition: max-height 0.3s ease, padding 0.3s;
  overflow: hidden;

  &.collapsed {
    max-height: 64px;
    padding: 16px 16px 0;
  }
}

.playlist-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  gap: 12px;
}
.left-head {
  display: flex;
  gap: 12px;
  align-items: center;
  h3 {
    margin: 0;
    color: var(--pink-core);
    font-weight: 800;
  }
}

.api-hint {
  color: var(--blue-glow);
  font-size: 0.9rem;
}

/* 搜索 */
.search-wrap {
  display: flex;
  align-items: center;
  gap: 8px;
  input {
    padding: 8px 12px;
    border-radius: 20px;
    border: 1px solid var(--pink-core);
    background: rgba(0, 0, 0, 0.3);
    color: var(--white-glow);
    width: 200px;
    outline: none;
    &:focus {
      border-color: var(--blue-glitch);
      box-shadow: 0 0 20px var(--blue-glitch);
    }
  }
  .clear {
    background: transparent;
    border: none;
    color: var(--white-glow);
    animation: cursorAnimation_link 1s infinite step-start;
    &:hover {
      color: var(--pink-core);
    }
  }
}

/* 列表区域 */
.list-area {
  position: relative;
  overflow: hidden;
}
.list-loading {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--blue-glow);
  padding: 10px 0;
}

.playlist {
  list-style: none;
  padding: 0;
  margin: 0;
  overflow-y: auto;
  max-height: 300px;
  display: block;
  &::-webkit-scrollbar {
    width: 4px;
  }
  &::-webkit-scrollbar-thumb {
    background: var(--pink-core);
    border-radius: 4px;
  }
  li {
    display: grid;
    grid-template-columns: 1fr auto;
    gap: 12px;
    align-items: center;
    padding: 10px;
    border-radius: 10px;
    animation: cursorAnimation_link 1s infinite step-start;
    transition: all 0.3s;
    border: 1px solid transparent;
    &:hover {
      transform: translateX(-6px);
      border-color: var(--pink-core);
      background: rgba(255, 140, 176, 0.05);
    }
    &.active {
      background: rgba(255, 140, 176, 0.1);
      border-color: var(--pink-core);
      box-shadow: 0 0 15px var(--pink-core);
    }

    .left-col {
      display: flex;
      gap: 12px;
      align-items: center;
      min-width: 0;
      .dot {
        width: 8px;
        height: 8px;
        border-radius: 50%;
        background: var(--pink-core);
        box-shadow: 0 0 10px var(--pink-core);
        flex-shrink: 0;
      }
      .title {
        font-weight: 700;
        color: var(--white-glow);
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        max-width: 100%;
      }
    }
    .right-col {
      .len {
        color: var(--blue-glow);
        font-weight: 700;
        min-width: 48px;
        text-align: right;
      }
    }
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
@media (max-width: 920px) {
  .player-main {
    padding-top: 80px;
  }
  .cover {
    height: 200px;
  }
  .list-section.collapsed {
    max-height: 60px;
  }
  .search-wrap input {
    width: 150px;
  }
  .core-status {
    display: none;
  }
  .ai-message {
    bottom: 10px;
    right: 10px;
    font-size: 11px;
    padding: 4px 10px;
  }
}

@media (max-width: 520px) {
  .btns {
    gap: 6px;
  }
  .volume input[type="range"] {
    width: 70px;
  }
  .playlist li .left-col .title {
    white-space: normal;
    -webkit-line-clamp: 2;
    display: -webkit-box;
    -webkit-box-orient: vertical;
  }
}
</style>
