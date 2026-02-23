<template>
  <div class="wiki-page">
    <!-- 故障光效背景层 + 数据粒子 + 扫描线 -->
    <div class="glitch-bg"></div>
    <div class="data-particles"></div>
    <div class="scan-line"></div>
  
    <!-- 背景轮播（保留原样） -->
    <div class="carousel">
      <div class="carousel-overlay"></div>
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>

    <header class="wiki-header">
      <div class="title">
        <h1 class="glitch-text" data-text="爱弥斯文本分享">爱弥斯文本分享</h1>
        <p class="subtitle">> 因与果，由此相衔。</p>
      </div>
      <div class="actions">
        <input
          v-model="search"
          class="search"
          placeholder="搜索标题或者标签..."
        />
        <button class="btn btn-new" @click="openCreate">新建词条</button>
      </div>
    </header>

    <main class="wiki-body">
      <div v-if="filteredEntries.length === 0" class="empty">
        > 没有找到匹配的词条 ✨_
      </div>

      <ul class="entry-list">
        <li v-for="entry in filteredEntries" :key="entry.id" class="entry-card">
          <div class="entry-card-inner">
            <!-- 故障边框 -->
            <div class="glitch-frame"></div>
            <!-- 全息扫描线 (hover时出现) -->
            <div class="hologram-scan"></div>

            <div class="entry-head">
              <div class="entry-meta" @click="openDetail(entry)">
                <div class="entry-title-wrap">
                  <h2 class="entry-title">{{ entry.title }}</h2>
                  <span class="entry-badge">#{{ entry.slug || "未设置" }}</span>
                </div>
                <div class="entry-info">
                  <span class="meta-item">作者：{{ entry.author }}</span>
                  <span class="meta-item"
                    >时间：{{ formatTime(entry.updatedAt) }}</span
                  >
                </div>
              </div>

              <div class="entry-actions">
                <button
                  class="like"
                  :class="{ liked: isLiked(entry.id) }"
                  :aria-pressed="isLiked(entry.id)"
                  @click.stop="toggleLike(entry.id)"
                >
                  <i class="heart"></i>
                  <span class="like-count">{{ entry.likes }}</span>
                </button>
                <div class="edit-delete" v-if="canEdit(entry.id)">
                  <button class="small" @click="openEdit(entry)">编辑</button>
                  <button class="small danger" @click="remove(entry.id)">
                    删除
                  </button>
                </div>
              </div>
            </div>
          </div>
        </li>
      </ul>
    </main>

    <!-- Edit/Create Modal -->
    <transition name="fade-zoom">
      <div class="modal-overlay" v-if="showModal">
        <div class="modal">
          <header class="modal-header">
            <h3>{{ editing ? "> 编辑词条_" : "> 新建词条_" }}</h3>
            <button class="close" @click="closeModal">✕</button>
          </header>
          <section class="modal-body">
            <label>
              标题
              <input v-model="form.title" placeholder="输入标题" />
            </label>

            <label>
              词条（短标签）
              <input
                v-model="form.slug"
                placeholder="比如：彩蛋、考据、点位等等"
              />
            </label>

            <label>
              作者
              <input v-model="form.author" placeholder="作者昵称" />
            </label>

            <label>
              内容
              <textarea
                v-model="form.content"
                rows="8"
                placeholder="在这里输入词条内容"
              ></textarea>
            </label>
          </section>
          <footer class="modal-footer">
            <button class="btn ghost" @click="closeModal">取消</button>
            <button class="btn" @click="submit">
              {{ editing ? "保存" : "创建" }}
            </button>
          </footer>
        </div>
      </div>
    </transition>

    <!-- Detail Modal -->
    <transition name="fade-zoom">
      <div class="modal-overlay" v-if="detailEntry">
        <div class="modal detail-modal">
          <header class="modal-header">
            <h3>> {{ detailEntry.title }}_</h3>
            <button class="close" @click="detailEntry = null">✕</button>
          </header>
          <section class="modal-body">
            <div class="detail-content">{{ detailEntry.content }}</div>
          </section>
        </div>
      </div>
    </transition>

    <!-- 浮动精灵 -->
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
// 脚本部分完全保持不变（功能接口不变）
import { ref, reactive, computed, onMounted, onUnmounted, nextTick } from "vue";
import { ElMessage } from "element-plus";
import {
  getWikiList,
  createWiki,
  updateWiki,
  deleteWiki,
  likeWiki,
} from "@/api/modules/wiki";
import { gsap } from "gsap";

// 本地存储自己创建的词条 ID
const LS_MY_WIKI_IDS = "yuzuriha:wiki:my_ids";
const myWikiIds: string[] = JSON.parse(
  localStorage.getItem(LS_MY_WIKI_IDS) || "[]"
);
const markAsMine = (id: string | number) => {
  if (!myWikiIds.includes(String(id))) {
    myWikiIds.push(String(id));
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
  }
};
const canEdit = (id: string | number) => myWikiIds.includes(String(id));

// 数据状态
const entries = ref<any[]>([]);

// 本地存储键
const LS_LIKED_IDS = "yuzuriha:wiki:liked_ids";
// 从 localStorage 读取已点赞 id 列表（字符串形式）
const likedIds = ref<string[]>(
  JSON.parse(localStorage.getItem(LS_LIKED_IDS) || "[]")
);

const showModal = ref(false);
const editing = ref(false);
const editingId = ref<string | number | null>(null);
const detailEntry = ref<any>(null);
const form = reactive({ title: "", slug: "", author: "", content: "" });
const search = ref("");

// 时间格式化
function formatTime(ts: string | number | null | undefined) {
  if (!ts) return "未知时间";
  const date = new Date(ts);
  if (isNaN(date.getTime())) return "未知时间";
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(
    2,
    "0"
  )}-${String(date.getDate()).padStart(2, "0")}`;
}

// 加载词条列表
async function loadEntries() {
  try {
    const res: any = await getWikiList();
    entries.value = res.data.map((e: any) => ({
      ...e,
      createdAt: e.createdAt || e.created_at,
      updatedAt: e.updatedAt || e.updated_at,
    }));
  } catch (err) {
    console.error(err);
    ElMessage.error("加载词条失败");
  }
}

// 打开/关闭弹窗
function openCreate() {
  editing.value = false;
  editingId.value = null;
  form.title = "";
  form.slug = "";
  form.author = "";
  form.content = "";
  showModal.value = true;
}
function openEdit(entry: any) {
  if (!canEdit(entry.id)) {
    ElMessage.warning("只有创建者可以编辑");
    return;
  }
  editing.value = true;
  editingId.value = entry.id;
  form.title = entry.title;
  form.slug = entry.slug;
  form.author = entry.author;
  form.content = entry.content;
  showModal.value = true;
}
function closeModal() {
  showModal.value = false;
}

const canSubmit = computed(() => form.title.trim() && form.content.trim());

// 提交
async function submit() {
  if (!canSubmit.value) {
    ElMessage.warning("请填写标题和内容");
    return;
  }
  const payload = {
    title: form.title.trim(),
    author: form.author.trim() || "匿名",
    content: form.content.trim(),
    slug: null,
  };
  if (form.slug.trim()) payload.slug = form.slug.trim();
  try {
    if (editing.value && editingId.value) {
      await updateWiki(editingId.value, payload);
      ElMessage.success("编辑成功");
    } else {
      const res: any = await createWiki(payload);
      markAsMine(res.data.id);
      editingId.value = res.data.id;
      ElMessage.success("创建成功");
    }
    showModal.value = false;
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("提交失败");
  }
}

// 删除
async function remove(id: string | number) {
  if (!canEdit(id)) {
    ElMessage.warning("只有创建者可以删除");
    return;
  }
  if (!confirm("确认删除该词条？此操作不可撤销")) return;
  try {
    await deleteWiki(id);
    const index = myWikiIds.indexOf(String(id));
    if (index !== -1) myWikiIds.splice(index, 1);
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
    ElMessage.success("删除成功");
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("删除失败");
  }
}

// 点赞
function persistLikedIds() {
  try {
    localStorage.setItem(LS_LIKED_IDS, JSON.stringify(likedIds.value));
  } catch (e) {
    console.warn("保存 likedIds 失败", e);
  }
}

// 判断是否已点赞
function isLiked(id: string | number) {
  return likedIds.value.includes(String(id));
}

// 点赞 / 取消点赞（乐观更新）
async function toggleLike(id: string | number) {
  const entry = entries.value.find((e) => e.id === id);
  if (!entry) return;

  const idStr = String(id);
  const wasLiked = likedIds.value.includes(idStr);

  // 乐观更新 UI
  if (wasLiked) {
    entry.likes = Math.max(0, (entry.likes || 0) - 1);
    likedIds.value = likedIds.value.filter((x) => x !== idStr);
  } else {
    entry.likes = (entry.likes || 0) + 1;
    likedIds.value.push(idStr);
  }
  persistLikedIds();

  try {
    const action = wasLiked ? "unlike" : "like";
    await likeWiki(id, action);
  } catch (err) {
    // 回滚
    console.error("toggleLike error", err);
    if (wasLiked) {
      entry.likes = (entry.likes || 0) + 1;
      if (!likedIds.value.includes(idStr)) likedIds.value.push(idStr);
    } else {
      entry.likes = Math.max(0, (entry.likes || 0) - 1);
      likedIds.value = likedIds.value.filter((x) => x !== idStr);
    }
    persistLikedIds();
    ElMessage.error("点赞失败，请稍后重试");
  }
}

// 详情弹窗
function openDetail(entry: any) {
  detailEntry.value = entry;
}

// 搜索过滤
const filteredEntries = computed(() => {
  const q = String(search.value || "")
    .trim()
    .toLowerCase();
  let list = entries.value;

  if (q) {
    list = list.filter(
      (e) =>
        (e.title || "").toLowerCase().includes(q) ||
        (e.slug || "").toLowerCase().includes(q)
    );
  }

  return [...list].sort((a, b) => (b.likes || 0) - (a.likes || 0));
});

// 轮播图
const pcModules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const mobileModules = import.meta.glob(
  "@/assets/images2/*.{jpg,png,jpeg,webp}",
  { eager: true }
);

const pcSrcs: string[] = Object.values(pcModules).map((m: any) => m.default);
const mobileSrcs: string[] = Object.values(mobileModules).map(
  (m: any) => m.default
);

function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

const randomFive = ref<string[]>([]);
const currentIndex = ref(0);
let timer: number;

function pickImages() {
  const isMobile = window.innerWidth < 768;
  const all = isMobile ? mobileSrcs : pcSrcs;
  randomFive.value = shuffle(all).slice(0, 5);
}

// 爱弥斯语录
const messages = [
  "数据流中检测到新的词条...",
  "这个条目很有价值。",
  "点击标题可以查看详情哦。",
  "感谢你的贡献。",
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
  loadEntries();
  pickImages();
  window.addEventListener("resize", pickImages);

  timer = window.setInterval(() => {
    if (randomFive.value.length > 0) {
      currentIndex.value = (currentIndex.value + 1) % randomFive.value.length;
    }
  }, 5000);

  // 初始化语录轮换
  msgInterval = window.setInterval(() => {
    nextMessage();
  }, 8000);

  // 初始化浮动精灵
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

onUnmounted(() => {
  clearInterval(timer);
  window.removeEventListener("resize", pickImages);
  if (msgInterval) clearInterval(msgInterval);
});
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

/* ===== 全局样式 ===== */
.wiki-page {
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
  color: var(--white-glow);
  padding: 16px;
  box-sizing: border-box;
  padding-top: 80px;
  background: var(--bg-deep);
  font-family: "JetBrains Mono", "Fira Code", "PingFang SC", monospace;
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



/* 轮播图（保留原样，增加叠加层） */
.carousel {
  position: absolute;
  inset: 0;
  z-index: -9;
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
    opacity: 0;
    transition: opacity 1s ease;
    filter: blur(1.5px) grayscale(6%);
    &.active {
      opacity: 1;
    }
  }
}

/* 头部 */
.wiki-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: rgba(26, 16, 40, 0.6);
  backdrop-filter: blur(10px);
  border: 1px solid var(--pink-core);
  border-radius: 12px;
  box-shadow: 0 0 30px var(--blue-glitch),
    inset 0 0 20px rgba(255, 140, 176, 0.1);
  flex-wrap: wrap;
  position: relative;
  z-index: 5;

  .title {
    h1 {
      margin: 0;
      font-size: 20px;
      font-weight: 900;
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
    .subtitle {
      font-size: 13px;
      color: var(--blue-glow);
      &::after {
        content: "_";
        animation: blink 1s infinite;
      }
    }
  }

  .actions {
    display: flex;
    gap: 8px;
    align-items: center;
    flex-wrap: wrap;
  }

  .search {
    padding: 8px 12px;
    border-radius: 20px;
    border: 1px solid var(--pink-core);
    background: rgba(0, 0, 0, 0.3);
    color: var(--white-glow);
    font-size: 13px;
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

  .btn-new {
    background: linear-gradient(135deg, var(--pink-core), var(--pink-light));
    color: #1a1028;
    border: none;
    border-radius: 20px;
    padding: 8px 16px;
    font-size: 14px;
    font-weight: 700;
    animation: cursorAnimation_link 1s infinite step-start;
    box-shadow: 0 0 15px var(--heart-glow);
    transition: all 0.3s;
    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 0 30px var(--pink-core), 0 0 60px var(--blue-glitch);
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

/* 主体 */
.wiki-body {
  margin-top: 20px;
  position: relative;
  z-index: 5;

  .empty {
    text-align: center;
    padding: 40px 16px;
    color: var(--blue-glow);
    &::after {
      content: "_";
      animation: blink 1s infinite;
    }
  }

  .entry-list {
    list-style: none;
    padding: 0;
    margin: 0;
    display: grid;
    gap: 16px;
  }
}

/* 卡片 */
.entry-card {
  perspective: 1000px;
  .entry-card-inner {
    position: relative;
    border-radius: 12px;
    padding: 16px;
    background: rgba(26, 16, 40, 0.6);
    backdrop-filter: blur(4px);
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
  }

  .entry-head {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: 8px;
    flex-wrap: wrap;

    .entry-meta {
      animation: cursorAnimation_link 1s infinite step-start;
      flex: 1;

      .entry-title-wrap {
        display: flex;
        align-items: center;
        gap: 8px;
        flex-wrap: wrap;
      }
      .entry-title {
        font-size: 18px;
        margin: 0;
        color: var(--pink-core);
        font-weight: 800;
        text-shadow: 0 0 8px var(--pink-core);
      }
      .entry-badge {
        display: inline-block;
        padding: 2px 8px;
        border-radius: 999px;
        background: rgba(110, 212, 255, 0.1);
        color: var(--blue-glow);
        font-size: 12px;
        border: 1px solid var(--blue-glitch);
      }
      .entry-info {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
        margin-top: 6px;
        .meta-item {
          font-size: 12px;
          color: var(--white-glow);
          background: rgba(255, 140, 176, 0.05);
          border-radius: 6px;
          padding: 2px 8px;
          border: 1px solid rgba(255, 140, 176, 0.2);
        }
      }
    }

    .entry-actions {
      display: flex;
      gap: 8px;
      align-items: center;
      flex-wrap: wrap;

      .like {
        background: transparent;
        border: 1px solid var(--pink-core);
        border-radius: 30px;
        display: flex;
        align-items: center;
        gap: 6px;
        padding: 6px 12px;
        animation: cursorAnimation_link 1s infinite step-start;
        transition: all 0.3s;
        .heart {
          width: 20px;
          height: 20px;
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
        .like-count {
          font-size: 13px;
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

      .edit-delete {
        display: flex;
        gap: 6px;
      }
      .small {
        padding: 6px 10px;
        border-radius: 8px;
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid var(--pink-core);
        color: var(--white-glow);
        font-size: 13px;
        animation: cursorAnimation_link 1s infinite step-start;
        transition: all 0.3s;
        &:hover {
          background: var(--pink-core);
          color: #1a1028;
          box-shadow: 0 0 15px var(--blue-glitch);
        }
      }
      .danger {
        border-color: #ff6b85;
        color: #ff6b85;
        &:hover {
          background: #ff6b85;
          color: #1a1028;
        }
      }
    }
  }
}

/* 模态框 */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(26, 16, 40, 0.9);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 16px;

  .modal {
    width: min(600px, 94%);
    max-height: 90vh;
    overflow-y: auto;
    background: rgba(26, 16, 40, 0.95);
    border: 2px solid var(--pink-core);
    border-radius: 16px;
    padding: 20px;
    box-shadow: 0 0 60px var(--blue-glitch),
      inset 0 0 30px rgba(255, 140, 176, 0.2);
    color: var(--white-glow);
    animation: fadeInUp 220ms ease;

    .modal-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding-bottom: 8px;
      h3 {
        margin: 0;
        color: var(--pink-core);
        &::after {
          content: "_";
          animation: blink 1s infinite;
        }
      }
      .close {
        background: transparent;
        border: 1px solid var(--pink-core);
        border-radius: 50%;
        width: 30px;
        height: 30px;
        display: flex;
        align-items: center;
        justify-content: center;
        color: var(--white-glow);
        animation: cursorAnimation_link 1s infinite step-start;
        transition: all 0.3s;
        &:hover {
          background: var(--pink-core);
          color: #1a1028;
          box-shadow: 0 0 15px var(--blue-glitch);
        }
      }
    }

    .modal-body {
      display: flex;
      flex-direction: column;
      gap: 12px;
      margin-top: 12px;
      .detail-content {
        white-space: pre-wrap;
        line-height: 1.6;
        color: var(--white-glow);
      }
      label {
        display: flex;
        flex-direction: column;
        gap: 4px;
        color: var(--blue-glow);
        input,
        textarea {
          background: rgba(0, 0, 0, 0.3);
          border: 1px solid var(--pink-light);
          border-radius: 8px;
          padding: 8px 12px;
          color: var(--white-glow);
          font-family: inherit;
          outline: none;
          &:focus {
            border-color: var(--blue-glitch);
            box-shadow: 0 0 20px var(--blue-glitch);
          }
        }
      }
    }

    .modal-footer {
      display: flex;
      justify-content: flex-end;
      gap: 12px;
      margin-top: 20px;
      .btn {
        padding: 8px 20px;
        border-radius: 20px;
        border: none;
        font-weight: 700;
        animation: cursorAnimation_link 1s infinite step-start;
        transition: all 0.3s;
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
        &.ghost {
          background: transparent;
          border: 1px solid var(--pink-core);
          color: var(--white-glow);
          box-shadow: none;
          &:hover {
            background: var(--pink-core);
            color: #1a1028;
          }
        }
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

/* 过渡动画 */
.fade-zoom-enter-active,
.fade-zoom-leave-active {
  transition: all 0.3s ease;
}
.fade-zoom-enter-from,
.fade-zoom-leave-to {
  opacity: 0;
  transform: scale(0.96);
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
@media (max-width: 720px) {

  .wiki-header {
    flex-direction: column;
    align-items: stretch;
  }
  .entry-list {
    display: flex;
    flex-direction: column;
  }
  .entry-actions {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }
  .entry-actions .like,
  .entry-actions .edit-delete {
    width: 100%;
    display: flex;
  }
  .entry-actions .like {
    justify-content: center;
  }
  .entry-actions .edit-delete {
    flex-direction: column;
  }
  .entry-actions .small {
    width: 100%;
  }
  .ai-message {
    bottom: 10px;
    right: 10px;
    font-size: 11px;
    padding: 4px 10px;
  }
}
</style>
