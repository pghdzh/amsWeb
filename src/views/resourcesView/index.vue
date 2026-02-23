<template>
  <div class="yuzuki-resources">
    <!-- 故障光效背景层 + 数据粒子 + 波形线条 -->
    <div class="glitch-bg"></div>
    <div class="wave-lines"></div>
    <div class="data-particles"></div>
    <div class="scan-line"></div>
    <!-- 核心能量状态 -->
    <div class="core-status">
      <span class="pulse-dot"></span>
      <span class="status-text">AI-EMS CORE: 98%</span>
    </div>

    <!-- 轮播图 -->
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

    <header class="hero">
      <div class="hero-inner">
        <h1 class="glitch-text" data-text="资源分享">资源分享</h1>
        <p class="subtitle">> 可自由上传关于爱弥斯的相关链接_</p>
      </div>
    </header>

    <main class="container">
      <section class="uploader" :class="{ collapsed: uploaderCollapsed }">
        <div class="uploader-head">
          <button
            class="toggle"
            @click="toggleUploader"
            :aria-expanded="!uploaderCollapsed"
          >
            <span v-if="uploaderCollapsed">展开上传区</span>
            <span v-else>收起上传区</span>
          </button>
        </div>

        <form
          @submit.prevent="addResource"
          class="upload-form"
          :aria-hidden="uploaderCollapsed"
        >
          <div class="row">
            <input
              v-model="form.title"
              type="text"
              placeholder="标题（必填，如果有解压码之类的也写这里吧）"
              aria-label="标题"
            />
            <input
              v-model="form.type"
              type="text"
              placeholder="链接类型(网页链接、b站视频、网盘链接等等)"
              aria-label="来源"
            />
          </div>

          <div class="row">
            <input
              v-model="form.uploader"
              type="text"
              placeholder="上传人（可选）"
              aria-label="上传人"
            />
            <input
              v-model="form.link"
              type="url"
              placeholder="链接(只输入网址不能有中文)"
              aria-label="链接"
            />
          </div>

          <div class="actions">
            <button type="submit" class="btn primary">上传</button>
          </div>
        </form>
      </section>

      <section class="list">
        <div class="list-header">
          <h2>资源列表（{{ resources.length }}）</h2>
          <div class="sort">
            <label>
              <span class="sort-icon">⌄</span>
              <select v-model="sortBy">
                <option value="time">按时间（新→旧）</option>
                <option value="likes">按点赞（高→低）</option>
              </select>
            </label>
          </div>
        </div>

        <ul class="items">
          <li v-for="item in sortedResources" :key="item.id" class="item">
            <div class="item-card">
              <!-- 故障边框 -->
              <div class="glitch-frame"></div>
              <!-- 全息扫描线 -->
              <div class="hologram-scan"></div>

              <a
                :href="item.link"
                target="_blank"
                rel="noopener noreferrer"
                class="title"
                >{{ item.title }}</a
              >

              <div class="meta">
                <div class="left">
                  <span class="uploader">{{ item.uploader || "匿名" }}</span>
                  <span class="dot">•</span>
                  <time :datetime="item.time">{{ formatTime(item.time) }}</time>
                </div>

                <div class="right">
                  <button
                    @click.prevent="handleLike(item)"
                    :aria-pressed="likedIds.has(String(item.id))"
                    class="like-btn"
                    :class="{ liked: likedIds.has(String(item.id)) }"
                  >
                    <i class="heart"></i>
                    <span class="count">{{ item.likes }}</span>
                  </button>

                  <span class="badge">{{ item.type }}</span>
                </div>
              </div>
            </div>
          </li>
        </ul>

        <p v-if="resources.length === 0" class="empty">
          > 目前没有资源，快来上传第一条吧！_
        </p>
      </section>
    </main>

    <footer class="foot">
      <span>> 提示：点击标题将直接跳转_</span>
    </footer>

    <!-- 浮动数据精灵（新增） -->
    <div class="floating-sprites">
      <div
        v-for="(sprite, i) in spriteList"
        :key="i"
        class="sprite"
        :style="{ top: sprite.top + 'px', left: sprite.left + 'px' }"
      ></div>
    </div>

    <!-- 爱弥斯核心语录（新增） -->
    <div class="ai-message" v-if="showAiMessage" @click="nextMessage">
      <span class="msg-label">爱弥斯说:</span>
      <span class="msg-content">{{ currentMessage }}</span>
      <span class="msg-cursor">_</span>
    </div>

    <!-- 上传成功闪光（新增） -->
    <div class="glitch-flash" v-if="flashVisible"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from "vue";
import {
  getResourceList,
  createResource,
  likeResource,
} from "@/api/modules/resource";
import { ElMessage } from "element-plus";
import { gsap } from "gsap";

// ---------- 类型定义 ----------
interface Resource {
  id: number | string;
  title: string;
  uploader?: string;
  time: string;
  likes: number;
  link: string;
  type: string;
  role_key?: string;
}

// ---------- 常量 ----------
const STORAGE_KEY = "fll_resources_v1";
const DEFAULT_ROLE = "ams";

// ---------- 响应式数据 ----------
const form = ref<{
  title: string;
  uploader: string;
  link: string;
  type: string;
}>({
  title: "",
  uploader: "",
  link: "",
  type: "",
});

const resources = ref<Resource[]>([]);
const likedIds = ref(new Set<string>());
const sortBy = ref<"time" | "likes">("time");
const uploaderCollapsed = ref(false);
const flashVisible = ref(false); // 上传闪光

// ---------- 爱弥斯语录 ----------
const messages = [
  "数据流中检测到新的资源...",
  "这个链接的量子态很稳定。",
  "点击标题可以直接跳转哦。",
  "感谢你为爱弥斯终端贡献资源。",
  "故障率低于阈值，系统正常。",
];
const currentMessage = ref(messages[0]);
const showAiMessage = ref(true);
let msgInterval: number;

function nextMessage() {
  const next = (messages.indexOf(currentMessage.value) + 1) % messages.length;
  currentMessage.value = messages[next];
}

// ---------- 浮动精灵 ----------
interface Sprite {
  top: number;
  left: number;
}
const spriteList = ref<Sprite[]>([]);

// ---------- 轮播图片 ----------
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

// ---------- 工具函数 ----------
function mapServerToLocal(row: any): Resource {
  return {
    id: row.id,
    title: row.title,
    uploader: row.uploader || "匿名",
    time: row.created_at || row.time || new Date().toISOString(),
    likes: row.likes ?? 0,
    link: row.link,
    type: row.storage_type || row.type || "other",
    role_key: row.role_key,
  };
}

async function loadResources() {
  try {
    const res: any = await getResourceList({
      role_key: DEFAULT_ROLE,
      page: 1,
      pageSize: 100,
    });
    if (res && res.success && Array.isArray(res.data)) {
      resources.value = res.data.map(mapServerToLocal);
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        try {
          const parsed = JSON.parse(raw);
          if (parsed.liked && Array.isArray(parsed.liked)) {
            parsed.liked.forEach((id: string) => likedIds.value.add(id));
          }
        } catch (e) {}
      }
      return;
    }
  } catch (err) {
    console.warn("拉取资源失败，使用本地缓存", err);
  }
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) {
      const parsed = JSON.parse(raw);
      if (parsed.liked && Array.isArray(parsed.liked)) {
        parsed.liked.forEach((id: string) => likedIds.value.add(id));
      }
    }
  } catch (e) {}
}

function saveLocalCache() {
  try {
    const liked = Array.from(likedIds.value);
    localStorage.setItem(STORAGE_KEY, JSON.stringify({ liked }));
  } catch (e) {}
}

function toggleUploader() {
  uploaderCollapsed.value = !uploaderCollapsed.value;
}

async function addResource() {
  const t = form.value.title.trim();
  const l = form.value.link.trim();
  if (!form.value.title.trim() || !form.value.link.trim()) {
    return ElMessage.warning("请填写完整信息");
  }
  if (!/^https?:\/\//i.test(l)) {
    return ElMessage.error("请输入正确的链接(https开头)");
  }
  try {
    const payload = {
      title: t,
      uploader: form.value.uploader.trim() || "匿名",
      link: l,
      storage_type: form.value.type,
      role_key: DEFAULT_ROLE,
    };
    const res: any = await createResource(payload);
    if (res && res.success && res.data) {
      const added = mapServerToLocal(res.data);
      resources.value.unshift(added);
      saveLocalCache();
      resetForm();
      ElMessage.success("上传成功");
      // 触发闪光
      triggerFlash();
      return;
    }
    ElMessage.error("上传失败");
  } catch (err) {
    console.warn("创建资源失败", err);
  }
}

function resetForm() {
  form.value.title = "";
  form.value.uploader = "";
  form.value.link = "";
  form.value.type = "";
}

async function handleLike(item: Resource) {
  const id = item.id;
  const wasLiked = likedIds.value.has(String(id));
  if (wasLiked) {
    likedIds.value.delete(String(id));
    item.likes = Math.max(0, item.likes - 1);
  } else {
    likedIds.value.add(String(id));
    item.likes++;
  }
  saveLocalCache();

  try {
    const action = wasLiked ? "unlike" : "like";
    const res: any = await likeResource(id, action);
    if (
      res &&
      res.success &&
      res.data &&
      typeof res.data.likes !== "undefined"
    ) {
      item.likes = res.data.likes;
    }
  } catch (err) {
    console.warn("点赞接口调用失败，回滚本地状态", err);
    if (wasLiked) {
      likedIds.value.add(String(id));
      item.likes++;
    } else {
      likedIds.value.delete(String(id));
      item.likes = Math.max(0, item.likes - 1);
    }
    saveLocalCache();
  }
}

function getFileType(link: string): string {
  if (link.includes("bilibili.com")) return "B站";
  if (link.includes("pan.baidu.com")) return "网盘";
  if (link.match(/\.(zip|rar|7z)$/i)) return "压缩包";
  return "链接";
}

function triggerFlash() {
  flashVisible.value = true;
  setTimeout(() => {
    flashVisible.value = false;
  }, 300);
}

const sortedResources = computed(() => {
  const arr = [...resources.value];
  if (sortBy.value === "time") {
    arr.sort((a, b) => +new Date(b.time) - +new Date(a.time));
  } else {
    arr.sort((a, b) => b.likes - a.likes);
  }
  return arr;
});

function formatTime(iso: string) {
  try {
    const d = new Date(iso);
    return new Intl.DateTimeFormat("zh-CN", {
      month: "2-digit",
      day: "2-digit",
      hour: "2-digit",
      minute: "2-digit",
    }).format(d);
  } catch (e) {
    return iso;
  }
}

// ---------- 生命周期 ----------
onMounted(() => {
  loadResources();
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  uploaderCollapsed.value = window.innerWidth <= 640;

  // 初始化浮动精灵
  const spriteCount = 5;
  const spriteSize = 40;
  spriteList.value = [];
  for (let i = 0; i < spriteCount; i++) {
    spriteList.value.push({
      left: Math.random() * (window.innerWidth - spriteSize),
      top: Math.random() * (window.innerHeight - spriteSize),
    });
  }
  // 给精灵添加 GSAP 动画
  nextTick(() => {
    const sprites = document.querySelectorAll<HTMLElement>(".sprite");
    sprites.forEach((el, idx) => {
      gsap.fromTo(
        el,
        { opacity: 0, scale: 0 },
        { opacity: 0.7, scale: 1, duration: 1, delay: idx * 0.2 }
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
            // 边界限制
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
      // 鼠标靠近闪避
      el.addEventListener("mouseenter", () => {
        gsap.killTweensOf(el);
        gsap.to(el, {
          x: "+=" + (Math.random() - 0.5) * 200,
          y: "+=" + (Math.random() - 0.5) * 200,
          duration: 0.8,
          ease: "back.out(2)",
          onComplete: () => {
            // 重新启动浮动动画（简化，不再递归）
          },
        });
      });
    });
  });

  // 启动语录轮换
  msgInterval = window.setInterval(() => {
    nextMessage();
  }, 8000);
});

onBeforeUnmount(() => {
  if (Imgtimer) clearInterval(Imgtimer);
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
@keyframes waveMove {
  0% {
    background-position: 0 0, 0 0;
  }
  100% {
    background-position: 0 40px, 40px 0;
  }
}

@keyframes flashFade {
  0% {
    opacity: 0.8;
    background: radial-gradient(
      circle at 50% 50%,
      var(--pink-core),
      var(--blue-glitch)
    );
  }
  100% {
    opacity: 0;
  }
}

/* ===== 全局样式 ===== */
.yuzuki-resources {
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
  color: var(--white-glow);
  display: flex;
  flex-direction: column;
  padding-top: 60px;
  font-family: "JetBrains Mono", "Fira Code", "PingFang SC", monospace;
  -webkit-font-smoothing: antialiased;
  background: var(--bg-deep);
  min-height: 100vh;
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

/* 波形线条（新增） */
.wave-lines {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 1;
  opacity: 0.15;
  background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 20px,
      var(--pink-core) 20px,
      var(--pink-core) 21px,
      transparent 21px,
      transparent 40px
    ),
    repeating-linear-gradient(
      90deg,
      transparent,
      transparent 20px,
      var(--blue-glitch) 20px,
      var(--blue-glitch) 21px,
      transparent 21px,
      transparent 40px
    );
  mask-image: radial-gradient(circle at 50% 50%, black 30%, transparent 70%);
  animation: waveMove 10s linear infinite;
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

/* 头部 */
.hero {
  padding: 18px 12px;
  background: rgba(26, 16, 40, 0.6);
  backdrop-filter: blur(10px) saturate(0.92);
  border-bottom: 1px solid var(--pink-core);
  position: relative;
  z-index: 5;
  .hero-inner {
    max-width: 1000px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    gap: 6px;
    h1 {
      margin: 0;
      font-size: 20px;
      font-weight: 900;
      letter-spacing: 0.6px;
      color: var(--pink-core);
      text-shadow: 0 0 10px var(--pink-core), 0 0 20px var(--blue-glitch);
      position: relative;
      &.glitch-text {
        animation: glitch 3s infinite;
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
    .subtitle {
      color: var(--blue-glow);
      font-size: 13px;
      &::after {
        content: "_";
        animation: blink 1s infinite;
      }
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

/* 主容器 */
.container {
  max-width: 1000px;
  margin: 16px auto;
  padding: 0 12px;
  width: 100%;
  box-sizing: border-box;
  z-index: 5;
  position: relative;
}

/* 上传器 */
.uploader {
  border-radius: 14px;
  padding: 0;
  box-shadow: 0 0 30px var(--blue-glitch),
    inset 0 0 20px rgba(255, 140, 176, 0.1);
  border: 1px solid var(--pink-core);
  background: rgba(26, 16, 40, 0.7);
  backdrop-filter: blur(10px);
  margin-bottom: 20px;

  .uploader-head {
    display: flex;
    justify-content: flex-end;
    padding: 10px 12px;
    .toggle {
      background: transparent;
      border: 1px solid var(--pink-core);
      color: var(--pink-core);
      padding: 6px 10px;
      border-radius: 8px;
      animation: cursorAnimation_link 1s infinite step-start;
      font-weight: 700;
      transition: all 0.3s;
      &:hover {
        background: var(--pink-core);
        color: #1a1028;
        box-shadow: 0 0 15px var(--blue-glitch);
      }
    }
  }

  .upload-form {
    padding: 14px;
    max-height: 1600px;
    overflow: hidden;
    transition: max-height 280ms ease, padding 280ms ease;

    .row {
      display: flex;
      gap: 8px;
      margin-bottom: 10px;

      input {
        flex: 1 1 0;
        padding: 10px 12px;
        border-radius: 10px;
        border: 1px solid var(--pink-light);
        font-size: 14px;
        background: rgba(0, 0, 0, 0.3);
        color: var(--white-glow);
        font-weight: 400;
        outline: none;
        transition: all 0.3s;
        &::placeholder {
          color: rgba(255, 255, 255, 0.4);
        }
        &:focus {
          border-color: var(--blue-glitch);
          box-shadow: 0 0 20px var(--blue-glitch);
        }
      }
    }

    .actions {
      display: flex;
      gap: 8px;
      align-items: center;

      .btn {
        padding: 8px 12px;
        border-radius: 10px;
        border: none;
        font-weight: 700;
        animation: cursorAnimation_link 1s infinite step-start;
        transition: all 0.3s;

        &.primary {
          background: linear-gradient(
            135deg,
            var(--pink-core),
            var(--pink-light)
          );
          color: #1a1028;
          box-shadow: 0 0 15px var(--heart-glow);
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 0 30px var(--pink-core), 0 0 60px var(--blue-glitch);
          }
        }
      }
    }
  }

  &.collapsed {
    .upload-form {
      max-height: 0;
      padding-top: 0;
      padding-bottom: 0;
    }
  }
}

/* 资源列表 */
.list {
  .list-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
    h2 {
      font-size: 16px;
      margin: 0;
      color: var(--pink-core);
      font-weight: 800;
      text-shadow: 0 0 8px var(--pink-core);
    }
    .sort {
      label {
        display: flex;
        align-items: center;
        gap: 4px;
        .sort-icon {
          color: var(--blue-glitch);
          font-size: 18px;
        }
        select {
          padding: 8px;
          border-radius: 8px;
          border: 1px solid var(--pink-core);
          background: rgba(26, 16, 40, 0.8);
          color: var(--white-glow);
          outline: none;
          animation: cursorAnimation_link 1s infinite step-start;
          &:focus {
            box-shadow: 0 0 15px var(--blue-glitch);
          }
        }
      }
    }
  }

  .items {
    list-style: none;
    padding: 0;
    margin: 0;
    max-height: 60vh;
    overflow: auto;
    &::-webkit-scrollbar {
      width: 4px;
    }
    &::-webkit-scrollbar-thumb {
      background: var(--pink-core);
      border-radius: 4px;
    }
  }

  .item {
    margin-bottom: 12px;
    perspective: 1000px;
    .item-card {
      position: relative;
      border-radius: 12px;
      padding: 12px;
      background: rgba(26, 16, 40, 0.6);

      border: 2px solid transparent;
      background-clip: padding-box;
      transition: all 0.4s;
      overflow: hidden;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
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

      .title {
        display: block;
        color: var(--white-glow);
        font-weight: 800;
        text-decoration: none;
        margin-bottom: 8px;
        font-size: 15px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        &:hover {
          color: var(--pink-core);
          text-shadow: 0 0 8px var(--pink-core);
        }
      }

      .meta {
        display: flex;
        justify-content: space-between;
        align-items: center;
        font-size: 13px;

        .left {
          display: flex;
          align-items: center;
          gap: 8px;
          .uploader {
            color: var(--blue-glow);
            font-weight: 700;
            padding: 6px;
            margin: auto 0;
          }
          .dot {
            opacity: 0.6;
          }
          time {
            color: var(--white-glow);
          }
        }

        .right {
          display: flex;
          align-items: center;
          gap: 8px;

          .like-btn {
            background: transparent;
            border: 1px solid var(--pink-core);
            border-radius: 30px;
            animation: cursorAnimation_link 1s infinite step-start;
            display: inline-flex;
            align-items: center;
            gap: 6px;
            padding: 4px 10px;
            transition: all 0.3s;
            .heart {
              width: 18px;
              height: 18px;
              background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="%23ff8cb0" stroke-width="2"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>')
                no-repeat center;
              background-size: contain;
              filter: drop-shadow(0 0 5px var(--pink-core));
            }
            &.liked .heart {
              background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%23ff8cb0" stroke="%23ff8cb0"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>')
                no-repeat center;
              background-size: contain;
              animation: pop 0.4s, pulse-glow 2s infinite;
            }
            .count {
              color: var(--white-glow);
              font-weight: bold;
            }
            &:hover {
              transform: scale(1.1);
              background: rgba(255, 140, 176, 0.2);
            }
          }
          @keyframes pop {
            0% {
              transform: scale(1);
            }
            50% {
              transform: scale(1.5);
            }
            100% {
              transform: scale(1);
            }
          }

          .badge {
            padding: 4px 8px;
            border-radius: 999px;
            font-size: 12px;
            font-weight: 700;
            background: rgba(110, 212, 255, 0.1);
            color: var(--blue-glow);
            border: 1px solid var(--blue-glitch);
          }
        }
      }
    }
  }

  .empty {
    text-align: center;
    color: var(--blue-glow);
    padding: 28px 0;
    &::after {
      content: "_";
      animation: blink 1s infinite;
    }
  }
}

/* 页脚 */
.foot {
  text-align: center;
  color: var(--pink-light);
  font-size: 12px;
  margin: 20px 0 40px;
  &::before {
    content: "> ";
    color: var(--blue-glitch);
  }
  &::after {
    content: "_";
    animation: blink 1s infinite;
  }
}

/* 浮动精灵（新增） */
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

/* 爱弥斯核心语录（新增） */
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

/* 上传成功闪光（新增） */
.glitch-flash {
  position: fixed;
  inset: 0;
  background: white;
  mix-blend-mode: overlay;
  z-index: 9999;
  pointer-events: none;
  animation: flashFade 0.3s ease-out;
}

/* 工具类 */
@keyframes borderRotate {
  0% {
    filter: hue-rotate(0deg);
  }
  100% {
    filter: hue-rotate(360deg);
  }
}

/* 响应式 */
@media (max-width: 640px) {
  .yuzuki-resources {
    padding-top: 80px;
  }
  .carousel1 {
    display: none;
  }
  .carousel2 {
    display: block;
  }
  .core-status {
    display: none;
  }
  .hero {
    padding: 12px 10px;
    .hero-inner h1 {
      font-size: 18px;
    }
    .subtitle {
      font-size: 12px;
    }
  }
  .container {
    padding: 0 14px;
  }
  .upload-form .row {
    flex-direction: column;
  }
  .items .item .title {
    white-space: normal;
  }
  .ai-message {
    bottom: 10px;
    right: 10px;
    font-size: 11px;
    padding: 4px 10px;
  }
}
</style>
