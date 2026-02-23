<template>
  <div class="chat-page">
    <!-- 故障光效背景层 + 数据粒子 + 动态数据流 -->
    <div class="glitch-bg"></div>
    <div class="data-stream"></div>
    <div class="data-particles"></div>
    <div class="scan-line"></div>

    <!-- 轮播背景图（保留） -->
    <div class="carousel carousel1" aria-hidden="true">
      <div class="carousel-overlay"></div>
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <div class="carousel carousel2" aria-hidden="true">
      <div class="carousel-overlay"></div>
      <img
        v-for="(src, idx) in randomFive2"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>

    <!-- 主终端区域 -->
    <div class="terminal-container">
      <!-- 顶部系统状态条（紧凑型） -->
      <div class="system-bar">
        <div class="system-item">
          <span class="label">总对话</span>
          <span class="value">{{ stats.totalChats }}</span>
        </div>
        <div class="system-item">
          <span class="label">首次</span>
          <span class="value">{{
            new Date(stats.firstTimestamp).toISOString().slice(0, 10)
          }}</span>
        </div>
        <div class="system-item">
          <span class="label">活跃</span>
          <span class="value">{{ stats.activeDates.length }}d</span>
        </div>
        <div class="system-item">
          <span class="label">今日</span>
          <span class="value">{{
            stats.dailyChats[new Date().toISOString().slice(0, 10)] || 0
          }}</span>
        </div>
        <button class="detail-btn" @click="showModal = true">详细</button>
      </div>

      <!-- 消息终端屏幕 -->
      <div class="terminal-screen">
        <div class="messages" ref="msgList">
          <transition-group name="msg" tag="div">
            <div
              v-for="msg in chatLog"
              :key="msg.id"
              :class="[
                'message',
                msg.role,
                { error: msg.isError, egg: msg.isEgg },
              ]"
            >
              <div class="avatar" :class="msg.role"></div>
              <div class="bubble">
                <div class="content" v-html="msg.text"></div>
              </div>
            </div>
            <div v-if="loading" class="message bot" key="loading">
              <div class="avatar bot"></div>
              <div class="bubble loading">
                正在思考中
                <span class="dots">
                  <span class="dot">.</span>
                  <span class="dot">.</span>
                  <span class="dot">.</span>
                </span>
              </div>
            </div>
          </transition-group>
        </div>
      </div>

      <!-- 命令行输入区 -->
      <div class="command-line">
        <span class="prompt">></span>
        <textarea
          v-model="input"
          placeholder="向爱弥斯提问…"
          :disabled="loading"
          @keydown="handleKeydown"
          rows="1"
        ></textarea>
        <div class="command-actions">
          <button
            type="button"
            class="action-btn"
            @click="clearChat"
            :disabled="loading"
            title="清空对话"
          >
            <span>✖</span>
          </button>
          <button
            type="button"
            class="action-btn"
            :disabled="!chatLog.length || loading"
            @click="copyChat"
          >
            <span>{{ copyButtonText }}</span>
          </button>
          <button
            type="submit"
            class="action-btn send"
            @click.prevent="sendMessage"
            :disabled="!input.trim() || loading"
          >
            发送
          </button>
          <button
            type="button"
            class="action-btn talkdata"
            @click="showModal = true"
            title="查看统计"
          >
            统计
          </button>
        </div>
      </div>
    </div>

    <!-- 详细统计弹窗（与之前相同） -->
    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-content">
        <h3>> 终端统计_</h3>
        <ul class="detail-list">
          <li>总对话次数：{{ stats.totalChats }}</li>
          <li>
            首次使用：{{
              new Date(stats.firstTimestamp).toISOString().slice(0, 10)
            }}
          </li>
          <li>活跃天数：{{ stats.activeDates.length }} 天</li>
          <li>
            今日对话：{{
              stats.dailyChats[new Date().toISOString().slice(0, 10)] || 0
            }}
            次
          </li>
          <li>总使用时长：{{ formatDuration(stats.totalTime) }}</li>
          <li>当前连续活跃：{{ stats.currentStreak }} 天</li>
          <li>最长连续活跃：{{ stats.longestStreak }} 天</li>
          <li>
            最活跃日：{{ mostActiveDayComputed }}（{{
              stats.dailyChats[mostActiveDayComputed] || 0
            }}
            次）
          </li>
        </ul>
        <button class="close-btn" @click="showModal = false">关闭</button>
      </div>
    </div>

    <!-- 浮动精灵（与之前相同） -->
    <div class="floating-sprites">
      <div
        v-for="(sprite, i) in spriteList"
        :key="i"
        class="sprite"
        :style="{ top: sprite.top + 'px', left: sprite.left + 'px' }"
      ></div>
    </div>

    <!-- 爱弥斯核心语录（与之前相同） -->
    <div class="ai-message" v-if="showAiMessage" @click="nextMessage">
      <span class="msg-label">爱弥斯说:</span>
      <span class="msg-content">{{ currentMessage }}</span>
      <span class="msg-cursor">_</span>
    </div>
  </div>
</template>

<script setup lang="ts">
// 脚本部分完全保持不变（功能接口不变）
import {
  reactive,
  ref,
  computed,
  onMounted,
  nextTick,
  watch,
  onBeforeUnmount,
} from "vue";
import { sendMessageToHui } from "@/api/deepseekApi";
import { gsap } from "gsap";

const STORAGE_KEY = "hui_chat_log";
const STORAGE_STATS_KEY = "hui_chat_stats";
const showModal = ref(false);

interface Stats {
  firstTimestamp: number;
  totalChats: number;
  activeDates: string[];
  dailyChats: Record<string, number>;
  currentStreak: number;
  longestStreak: number;
  totalTime: number;
}

const defaultStats: Stats = {
  firstTimestamp: Date.now(),
  totalChats: 0,
  activeDates: [],
  dailyChats: {},
  currentStreak: 0,
  longestStreak: 0,
  totalTime: 0,
};

function loadStats(): Stats {
  const saved = localStorage.getItem(STORAGE_STATS_KEY);
  if (saved) {
    try {
      const parsed = JSON.parse(saved);
      return { ...defaultStats, ...parsed };
    } catch {
      console.warn("加载统计数据失败，使用默认值");
    }
  }
  return { ...defaultStats };
}

function saveStats() {
  localStorage.setItem(STORAGE_STATS_KEY, JSON.stringify(stats));
}

function updateActive(date: string) {
  if (!stats.activeDates.includes(date)) {
    stats.activeDates.push(date);
    updateStreak();
    saveStats();
  }
}
function updateStreak() {
  const dates = [...stats.activeDates].sort();
  let curr = 0,
    max = stats.longestStreak,
    prevTs = 0;
  const todayStr = new Date().toISOString().slice(0, 10);
  dates.forEach((d) => {
    const ts = new Date(d).getTime();
    if (prevTs && ts - prevTs === 86400000) curr++;
    else curr = 1;
    max = Math.max(max, curr);
    prevTs = ts;
  });
  stats.currentStreak = dates[dates.length - 1] === todayStr ? curr : 0;
  stats.longestStreak = max;
  saveStats();
}

function updateDaily(date: string) {
  stats.dailyChats[date] = (stats.dailyChats[date] || 0) + 1;
  saveStats();
}

const mostActiveDayComputed = computed(() => {
  let day = "",
    max = 0;
  for (const [d, c] of Object.entries(stats.dailyChats)) {
    if (c > max) {
      max = c;
      day = d;
    }
  }
  return day || new Date().toISOString().slice(0, 10);
});

function formatDuration(ms: number): string {
  const totalMin = Math.floor(ms / 60000);
  const h = Math.floor(totalMin / 60);
  const m = totalMin % 60;
  return h ? `${h} 小时 ${m} 分钟` : `${m} 分钟`;
}

const stats = reactive<Stats>(loadStats());
const sessionStart = Date.now();

interface ChatMsg {
  id: number;
  role: "user" | "bot";
  text: string;
  isError?: boolean;
  isEgg?: boolean;
}

const chatLog = ref<ChatMsg[]>(loadChatLog());
const input = ref("");
const loading = ref(false);
const msgList = ref<HTMLElement>();

async function sendMessage() {
  if (!input.value.trim()) return;
  if (stats.totalChats === 0 && !localStorage.getItem(STORAGE_STATS_KEY)) {
    stats.firstTimestamp = Date.now();
    saveStats();
  }
  const date = new Date().toISOString().slice(0, 10);
  stats.totalChats++;
  updateActive(date);
  updateDaily(date);
  saveStats();

  const userText = input.value;
  chatLog.value.push({
    id: Date.now(),
    role: "user",
    text: userText,
  });
  input.value = "";
  loading.value = true;

  try {
    const history = chatLog.value.filter((msg) => !msg.isEgg && !msg.isError);
    const botReply = await sendMessageToHui(userText, history);
    if (botReply == "error") {
      chatLog.value.push({
        id: Date.now() + 2,
        role: "bot",
        text: "API余额耗尽了，去b站提醒我充钱吧",
        isError: true,
      });
    } else {
      chatLog.value.push({
        id: Date.now() + 1,
        role: "bot",
        text: botReply,
      });
    }
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
    await scrollToBottom();
  }
}

function handleKeydown(e: KeyboardEvent) {
  if (e.key === "Enter") sendMessage();
}

function clearChat() {
  if (confirm("确定要清空全部对话吗？")) {
    chatLog.value = [
      {
        id: Date.now(),
        role: "bot",
        text: "嘿~你来啦。今天想听我唱歌，还是想玩拉海洛方块？",
      },
    ];
    localStorage.removeItem(STORAGE_KEY);
  }
}

function loadChatLog(): ChatMsg[] {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      return JSON.parse(saved);
    } catch (e) {
      console.error("chatLog 解析失败：", e);
    }
  }
  return [
    {
      id: Date.now(),
      role: "bot",
      text: "嘿~你来啦。今天想听我唱歌，还是想玩拉海洛方块？",
    },
  ];
}

async function scrollToBottom() {
  await nextTick();
  if (msgList.value) {
    msgList.value.scrollTop = msgList.value.scrollHeight;
  }
}

const copyButtonText = ref("复制");
let _copyTimer: number | null = null;

function stripHtml(html = ""): string {
  if (typeof document === "undefined") {
    return html.replace(/<br\s*\/?>/gi, "\n").replace(/<[^>]+>/g, "");
  }
  const div = document.createElement("div");
  const withBreaks = String(html).replace(/<br\s*\/?>/gi, "\n");
  div.innerHTML = withBreaks;
  return div.textContent || div.innerText || "";
}

function buildChatPlainText(): string {
  return chatLog.value
    .map((msg) => {
      const time = new Date(msg.id).toLocaleString();
      const who = msg.role === "user" ? "你" : "爱弥斯";
      const text = stripHtml(msg.text);
      return `[${time}] ${who}: ${text}`;
    })
    .join("\n\n");
}

function fallbackCopyText(text: string) {
  const textarea = document.createElement("textarea");
  textarea.value = text;
  textarea.style.position = "fixed";
  textarea.style.left = "-9999px";
  textarea.style.top = "0";
  document.body.appendChild(textarea);
  textarea.select();
  textarea.setSelectionRange(0, textarea.value.length);
  const ok = document.execCommand("copy");
  document.body.removeChild(textarea);
  if (!ok) throw new Error("execCommand copy failed");
}

async function copyChat() {
  const text = buildChatPlainText();
  if (!text) {
    copyButtonText.value = "无内容可复制";
    clearTimeout(_copyTimer as number);
    _copyTimer = window.setTimeout(() => (copyButtonText.value = "复制"), 1600);
    return;
  }

  try {
    if (navigator.clipboard && navigator.clipboard.writeText) {
      await navigator.clipboard.writeText(text);
    } else {
      fallbackCopyText(text);
    }
    copyButtonText.value = "已复制";
  } catch (err) {
    try {
      fallbackCopyText(text);
      copyButtonText.value = "已复制";
    } catch (e) {
      console.error("复制失败：", e);
      copyButtonText.value = "复制失败";
    }
  } finally {
    clearTimeout(_copyTimer as number);
    _copyTimer = window.setTimeout(() => (copyButtonText.value = "复制"), 1600);
  }
}

watch(
  chatLog,
  async () => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(chatLog.value));
    await scrollToBottom();
  },
  { deep: true }
);

function handleBeforeUnload() {
  stats.totalTime += Date.now() - sessionStart;
  saveStats();
}

// 轮播图
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
let Imgtimer: number | undefined;

// 爱弥斯语录
const messages = [
  "数据流中检测到新的对话...",
  "你的消息我已接收。",
  "要听听我的秘密吗？",
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
  scrollToBottom();
  window.addEventListener("beforeunload", handleBeforeUnload);
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);

  msgInterval = window.setInterval(() => {
    nextMessage();
  }, 8000);

  const spriteCount = 6;
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
        { opacity: 0.6, scale: 1, duration: 1, delay: idx * 0.1 }
      );
      gsap.to(el, {
        y: "+=30",
        x: "+=15",
        duration: 3 + Math.random() * 2,
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
  window.removeEventListener("beforeunload", handleBeforeUnload);
  if (Imgtimer) clearInterval(Imgtimer);
  if (_copyTimer) clearTimeout(_copyTimer);
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
@keyframes data-stream-flow {
  0% {
    background-position: 0 0;
  }
  100% {
    background-position: 0 100px;
  }
}

/* ===== 全局样式 ===== */
.chat-page {
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
  padding-top: 64px;
  min-height: 100vh;
  font-family: "JetBrains Mono", "Fira Code", "PingFang SC", monospace;
  color: var(--white-glow);
  display: flex;
  flex-direction: column;
  background: var(--bg-deep);
  position: relative;
  overflow-x: hidden;
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

/* 动态数据流 */
.data-stream {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 1;
  opacity: 0.1;
  background: repeating-linear-gradient(
    45deg,
    transparent,
    transparent 20px,
    var(--pink-core) 20px,
    var(--pink-core) 22px,
    transparent 22px,
    transparent 40px
  );
  animation: data-stream-flow 10s linear infinite;
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

/* 轮播图 */
.carousel {
  position: absolute;
  inset: 0;
  z-index: 0;
  pointer-events: none;

  .carousel-overlay {
    position: absolute;
    inset: 0;
    background: linear-gradient(
      180deg,
      rgba(26, 16, 40, 0.4) 0%,
      rgba(40, 30, 48, 0.6) 100%
    );
    mix-blend-mode: overlay;
    z-index: 2;
    pointer-events: none;
  }

  .carousel-image {
    position: absolute;
    width: 100%;
    height: 100%;
    object-fit: cover;
    filter: blur(0.8px) saturate(0.75) brightness(0.7);
    opacity: 0;
    transition: opacity 1.5s ease;
    &.active {
      opacity: 1;
    }
  }
}
.carousel2 {
  display: none;
}

/* 终端容器（新布局） */
.terminal-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  max-width: 1000px;
  margin: 0 auto;
  width: 100%;
  padding: 0 20px;
  position: relative;
  z-index: 5;
}

/* 顶部系统状态条 */
.system-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(26, 16, 40, 0.7);

  border: 1px solid var(--pink-core);
  border-radius: 30px;
  padding: 8px 16px;
  margin: 16px 0 12px;
  box-shadow: 0 0 20px var(--blue-glitch);

  .system-item {
    display: flex;
    align-items: baseline;
    gap: 4px;
    .label {
      font-size: 11px;
      color: var(--blue-glow);
      text-transform: uppercase;
    }
    .value {
      color: var(--pink-core);
      font-weight: 700;
      font-size: 14px;
      text-shadow: 0 0 6px var(--pink-core);
    }
  }

  .detail-btn {
    background: transparent;
    border: 1px solid var(--pink-core);
    border-radius: 20px;
    color: var(--white-glow);
    padding: 4px 12px;
    animation: cursorAnimation_link 1s infinite step-start;
    font-size: 12px;
    transition: all 0.3s;
    &:hover {
      background: var(--pink-core);
      color: #1a1028;
      box-shadow: 0 0 15px var(--blue-glitch);
    }
  }
}

/* 终端屏幕 */
.terminal-screen {
  flex: 1;
  background: rgba(26, 16, 40, 0.4);
  backdrop-filter: blur(8px);
  border: 1px solid var(--pink-core);
  border-radius: 16px;
  padding: 20px;
  margin-bottom: 16px;
  box-shadow: 0 0 30px var(--blue-glitch),
    inset 0 0 20px rgba(255, 140, 176, 0.1);
  overflow: hidden;
}

.messages {
  height: 100%;
  overflow-y: auto;
  overscroll-behavior: contain;
  scroll-behavior: smooth;
  max-height: 60vh;
  padding-right: 8px;

  &::-webkit-scrollbar {
    width: 4px;
  }
  &::-webkit-scrollbar-thumb {
    background: var(--pink-core);
    border-radius: 4px;
  }
}

/* 消息气泡 - 爱弥斯风格（与之前相同，但微调） */
.message {
  display: flex;
  align-items: flex-start;
  margin-bottom: 16px;

  &.user {
    flex-direction: row-reverse;
  }

  .avatar {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    margin: 0 10px;
    background-size: cover;
    background-position: center;
    flex-shrink: 0;
    border: 2px solid var(--pink-core);
    box-shadow: 0 0 15px var(--blue-glitch);

    &.bot {
      background-image: url("@/assets/avatar/ams.png");
      animation: pulse-glow 3s infinite;
    }
    &.user {
      background: linear-gradient(135deg, var(--pink-core), var(--blue-glitch));
    }
  }

  .bubble {
    max-width: 70%;
    background: rgba(26, 16, 40, 0.8);
    backdrop-filter: blur(4px);
    border: 2px solid transparent;
    padding: 12px 16px;
    border-radius: 16px;
    line-height: 1.6;
    position: relative;
    transition: all 0.3s;

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
      opacity: 0;
      transition: opacity 0.3s;
    }
    &:hover::before {
      opacity: 1;
    }

    .dots {
      display: inline-flex;
      margin-left: 6px;
      .dot {
        opacity: 0;
        animation: blink 1s infinite;
        color: var(--blue-glow);
        &:nth-child(2) {
          animation-delay: 0.18s;
        }
        &:nth-child(3) {
          animation-delay: 0.36s;
        }
      }
    }
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

/* 命令行输入区（新） */
.command-line {
  display: flex;
  align-items: center;
  background: rgba(26, 16, 40, 0.8);
  backdrop-filter: blur(10px);
  border: 1px solid var(--pink-core);
  border-radius: 40px;
  padding: 8px 16px;
  margin-bottom: 20px;
  box-shadow: 0 0 30px var(--blue-glitch);

  .prompt {
    color: var(--pink-core);
    font-size: 18px;
    font-weight: bold;
    margin-right: 10px;
    animation: blink 1s infinite;
  }

  textarea {
    flex: 1;
    background: transparent;
    border: none;
    color: var(--white-glow);
    font-size: 15px;
    line-height: 1.5;
    outline: none;
    resize: none;
    overflow: hidden;
    min-height: 40px;
    max-height: 100px;
    padding: 8px 0;
    &::placeholder {
      color: rgba(255, 255, 255, 0.3);
    }
  }

  .command-actions {
    display: flex;
    gap: 8px;
    margin-left: 10px;

    .action-btn {
      background: transparent;
      border: 1px solid var(--pink-core);
      border-radius: 20px;
      color: var(--white-glow);
      padding: 6px 12px;
      animation: cursorAnimation_link 1s infinite step-start;
      font-size: 13px;
      transition: all 0.3s;
      min-width: 50px;

      &:hover:not(:disabled) {
        background: var(--pink-core);
        color: #1a1028;
        box-shadow: 0 0 15px var(--blue-glitch);
        transform: translateY(-2px);
      }
      &:disabled {
        opacity: 0.4;
        animation: cursorAnimation_disabled 1s infinite step-start;
      }

      &.send {
        background: linear-gradient(
          135deg,
          var(--pink-core),
          var(--pink-light)
        );
        color: #1a1028;
        font-weight: bold;
        border: none;
        box-shadow: 0 0 15px var(--heart-glow);
        &:hover:not(:disabled) {
          box-shadow: 0 0 30px var(--pink-core), 0 0 60px var(--blue-glitch);
        }
      }
    }
    .talkdata {
      display: none;
    }
  }
}

/* 模态框（与之前相同） */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(26, 16, 40, 0.9);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: 16px;

  .modal-content {
    width: 380px;
    max-width: 100%;
    background: rgba(26, 16, 40, 0.95);
    border: 2px solid var(--pink-core);
    border-radius: 14px;
    padding: 20px;
    color: var(--white-glow);
    box-shadow: 0 0 60px var(--blue-glitch),
      inset 0 0 30px rgba(255, 140, 176, 0.2);
    animation: fadeInUp 220ms ease;

    h3 {
      margin: 0 0 12px 0;
      font-size: 18px;
      font-weight: 800;
      text-align: center;
      color: var(--pink-core);
      &::after {
        content: "_";
        animation: blink 1s infinite;
      }
    }

    .detail-list {
      list-style: none;
      padding: 0;
      margin: 12px 0 18px;
      line-height: 1.6;
      font-size: 14px;
      li {
        margin-bottom: 8px;
        padding-left: 6px;
        border-bottom: 1px dashed rgba(255, 140, 176, 0.2);
      }
    }

    .close-btn {
      display: block;
      margin: 0 auto;
      padding: 8px 20px;
      background: linear-gradient(135deg, var(--pink-core), var(--pink-light));
      color: #1a1028;
      border: none;
      border-radius: 10px;
      animation: cursorAnimation_link 1s infinite step-start;
      font-weight: 700;
      box-shadow: 0 0 15px var(--heart-glow);
      transition: all 0.3s;
      &:hover {
        transform: translateY(-3px);
        box-shadow: 0 0 30px var(--pink-core), 0 0 60px var(--blue-glitch);
      }
    }
  }
}

/* 浮动精灵（与之前相同） */
.floating-sprites {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 5;
}
.sprite {
  position: absolute;
  width: 40px;
  height: 40px;
  background: radial-gradient(
    circle at 30% 30%,
    var(--pink-core),
    var(--blue-glitch)
  );
  border-radius: 50%;
  filter: blur(2px) drop-shadow(0 0 10px var(--pink-glow));
  opacity: 0.6;
  pointer-events: auto;
  transition: filter 0.3s;
  &:hover {
    filter: blur(0) drop-shadow(0 0 20px var(--pink-core))
      drop-shadow(0 0 40px var(--blue-glow));
    opacity: 0.9;
  }
}

/* 爱弥斯语录（与之前相同） */
.ai-message {
  position: fixed;
  bottom: 0px;
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

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 响应式 */
@media (max-width: 768px) {
  .system-bar {
    flex-wrap: wrap;
    gap: 8px;
    .system-item {
      .label {
        font-size: 10px;
      }
      .value {
        font-size: 12px;
      }
    }
  }
  .command-line {
    flex-direction: column;
    align-items: stretch;
    border-radius: 20px;
    padding: 12px;
    .prompt {
      display: none;
    }
    textarea {
      width: 100%;
      margin-bottom: 8px;
    }
    .command-actions {
      margin-left: 0;
      justify-content: space-around;
    }
  }
  .carousel1 {
    display: none;
  }
  .carousel2 {
    display: block;
  }
  .ai-message {
    bottom: 0px;
    right: 10px;
    font-size: 11px;
    padding: 4px 10px;
  }
  .command-line {
    .command-actions {
      .talkdata {
        display: block;
      }
    }
  }
}
</style>
