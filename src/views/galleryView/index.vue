<template>
  <div class="gallery-container">
    <!-- 故障光效背景层：增强电子幽灵氛围 -->
    <div class="glitch-bg"></div>
    <!-- 新增：漂浮的数据粒子层 -->
    <div class="data-particles"></div>
    <!-- 新增：动态故障光栅 -->
    <div class="scan-line"></div>
    <!-- 新增：爱弥斯核心能量读数（装饰） -->
    <div class="core-status">
      <span class="pulse-dot"></span>
      <span class="status-text">AI-EMS CORE: 98%</span>
    </div>

    <button class="upload-btn" @click="openUploadModal">上传图片</button>

    <section class="gallery section">
      <div class="sort-controls">
        <button @click="toggleSort" class="sort-btn">
          <span class="btn-glitch-text">{{
            sortBy === "like_count" ? "点赞量" : "最新上传"
          }}</span>
          <span class="btn-subtext">排序</span>
        </button>
      </div>
      <div class="gallery-grid">
        <div
          v-for="(img, index) in images"
          :key="img.id"
          class="card"
          @click="openLightbox(index)"
          ref="cards"
        >
          <div class="card-inner">
            <!-- 故障边框 -->
            <div class="glitch-frame"></div>
            <!-- 新增：全息扫描线 (hover时出现) -->
            <div class="hologram-scan"></div>
            <img
              :src="img.src"
              :alt="img.alt"
              loading="lazy"
              @load="onImageLoad($event)"
            />
            <div class="overlay">
              <span> > 查看大图_</span>
            </div>
            <button class="like-btn" @click.stop="handleLike(img)">
              <i class="heart" :class="{ liked: img.liked }"></i>
              <span class="like-count">{{ img.likeCount }}</span>
            </button>
          </div>
        </div>
      </div>
      <!-- sentinel：用于触发无限滚动 -->
      <div ref="sentinel" class="sentinel"></div>
      <!-- 可选：加载中/结束提示，风格化为系统载入中 -->
      <div class="loading" v-if="loading">
        <span class="blink">> 加载数据中...</span>
      </div>
      <div class="finished" v-if="finished">
        <span>> 已到达数据终端_</span>
      </div>
    </section>

    <!-- 排行榜面板 - 设计为“爱弥斯的电子终端” -->
    <aside class="ranking-panel" :class="{ collapsed: !expanded }">
      <div class="panel-header" @click="expanded = !expanded">
        <h3 class="ranking-title">终端排行榜</h3>
        <span class="stats-badge">共{{ imgTotal }}张</span>
        <span class="toggle-icon">{{ expanded ? "▾" : "▸" }}</span>
      </div>
      <transition name="fade">
        <ul v-if="expanded" class="ranking-list">
          <li
            v-for="(item, idx) in rankingList"
            :key="idx"
            class="ranking-item"
            :class="`rank-${idx + 1}`"
          >
            <span class="rank">{{ idx + 1 }}</span>
            <span class="name">{{ item.nickname }}</span>
            <span class="count">{{ item.count }} 张</span>
          </li>
        </ul>
      </transition>
    </aside>

    <!-- Lightbox Modal -->
    <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
      <span class="close" @click="closeLightbox">✕</span>
      <span class="prev" @click.stop="prevImage">‹</span>
      <img :src="images[currentIndex].src" :alt="images[currentIndex].alt" />
      <span class="next" @click.stop="nextImage">›</span>
    </div>

    <!-- 上传弹窗 - 电子幽灵的“登陆界面” -->
    <div
      v-if="uploadModalOpen"
      class="upload-modal-overlay"
      @click.self="closeUploadModal"
    >
      <div class="upload-modal">
        <h3>> 上传终端_</h3>
        <div class="tip-container">
          <ul class="tips-list">
            <li>
              审核规则： 1.不要色情倾向（不要露三点，我怕被封）
              2.要我能认出是爱弥斯。
            </li>
            <li>
              由于没有用户系统，我这边不好做审核反馈，但只要显示上传成功，我这边肯定能收到。
            </li>
            <li>
              如果图片数量较多请在b站私信联系我给我网盘链接，因为我云服务器比较小一次性上传太多图片可能会导致上传不上，感谢理解。
            </li>
            <li>
              因为审核上传一次比较麻烦，所以审核时间不定，最晚一周，感谢谅解。
            </li>
          </ul>
        </div>
        <p class="stats">
          今日已上传：<strong>{{ uploadedToday }}</strong> 张，
          剩余可上传：<strong>{{ remaining }}</strong> 张
        </p>
        <label>
          <span class="label-icon">></span> 昵称：
          <input v-model="nickname" type="text" placeholder="请输入昵称" />
        </label>
        <label>
          <span class="label-icon">></span> 选择图片（最多
          {{ remaining }} 张）：
          <input
            ref="fileInput"
            type="file"
            multiple
            accept="image/*"
            @change="handleFileSelect"
          />
        </label>
        <p class="tip" v-if="selectedFiles.length">
          已选 {{ selectedFiles.length }} 张
        </p>
        <div class="modal-actions">
          <button :disabled="!canSubmit || isUploading" @click="submitUpload">
            {{ isUploading ? "上传中..." : "立即上传" }}
          </button>
          <button class="cancel" @click="closeUploadModal">取消</button>
        </div>
      </div>
    </div>

    <!-- 浮动小宠物 - 爱弥斯的“电子影像”分身 -->
    <div class="floating-chibis">
      <img
        v-for="(pet, i) in chibiList"
        :key="i"
        :src="pet.src"
        :style="{ top: pet.top + 'px', left: pet.left + 'px' }"
        class="chibi-img"
      />
    </div>
  </div>
</template>

<script lang="ts" setup>
// 脚本部分完全保持不变（功能接口不变）
import { ref, onMounted, computed, nextTick, onBeforeUnmount } from "vue";
import { uploadImages } from "@/api/modules/images";
import { getRankingList } from "@/api/modules/ranking";
import { gsap } from "gsap";
import { getImagesLikesList, likeImage } from "@/api/modules/imagesLikes";
import { debounce } from "lodash";

const sortBy = ref<"uploaded_at" | "like_count">("like_count");
const order = ref<"asc" | "desc">("desc");
function toggleSort() {
  if (sortBy.value === "uploaded_at") {
    sortBy.value = "like_count";
    order.value = "desc";
  } else {
    sortBy.value = "uploaded_at";
    order.value = "desc";
  }
  pageImage.value = 1;
  images.value = [];
  finished.value = false;
  window.scrollTo(0, 0);
  loadNextPage();
}
function getLikedIds(): number[] {
  const data = localStorage.getItem("likedImageIds");
  return data ? JSON.parse(data) : [];
}
function setLikedIds(ids: number[]) {
  localStorage.setItem("likedImageIds", JSON.stringify(ids));
}
async function handleLike(img: ImageItem) {
  if (img.liked) return;
  try {
    await likeImage(img.id);
    img.likeCount += 1;
    img.liked = true;
    const likedIds = getLikedIds();
    likedIds.push(img.id);
    setLikedIds(likedIds);
  } catch (error) {
    console.error("点赞失败", error);
    alert("点赞失败，请稍后重试");
  }
}
interface ImageItem {
  src: string;
  alt: string;
  likeCount: number;
  id: number;
  liked: Boolean;
}
interface RankingItem {
  id?: number;
  nickname: string;
  count: number;
}
const rankingList = ref<RankingItem[]>([]);
const expanded = ref(true);
const page = 1;
const pageSize = 99;
const fetchRanking = async () => {
  const res = await getRankingList({
    page,
    pageSize,
    character_key: "ams",
  });
  if (res.success) {
    rankingList.value = res.data;
  } else {
    console.error("获取排行榜失败", res.message);
  }
};
const images = ref<ImageItem[]>([]);
const pageImage = ref(1);
const limit = ref(10);
const loading = ref(false);
const finished = ref(false);
const sentinel = ref<HTMLElement | null>(null);
const observerCard = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add("visible");
        observerCard.unobserve(entry.target);
      }
    });
  },
  { threshold: 0.1 }
);
async function observeNewCards(startIndex = 0) {
  await nextTick();
  const cards = document.querySelectorAll<HTMLElement>(".card");
  for (let i = startIndex; i < cards.length; i++) {
    observerCard.observe(cards[i]);
  }
}
const imgTotal = ref(0);
async function loadNextPage() {
  if (loading.value || finished.value) return;
  loading.value = true;
  try {
    const res = await getImagesLikesList({
      page: pageImage.value,
      limit: limit.value,
      sortBy: sortBy.value,
      character_key: "ams",
      order: order.value,
    });
    imgTotal.value = res.total;
    const likedIds = getLikedIds();
    const list = (
      res.images as Array<{ url: string; like_count: number; id: number }>
    ).map((item) => ({
      src: item.url,
      alt: "",
      likeCount: item.like_count,
      id: item.id,
      liked: likedIds.includes(item.id),
    }));
    if (list.length === 0) {
      finished.value = true;
      return;
    }
    const oldLength = images.value.length;
    const existingIds = new Set(images.value.map((i) => i.id));
    const filtered = list.filter((item) => !existingIds.has(item.id));
    images.value.push(...filtered);
    pageImage.value++;
    observeNewCards(oldLength);
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}
const debouncedLoad = debounce(
  () => {
    loadNextPage();
  },
  200,
  { leading: true, trailing: false }
);
const lightboxOpen = ref(false);
const currentIndex = ref(0);
function openLightbox(index: number) {
  currentIndex.value = index;
  lightboxOpen.value = true;
}
function closeLightbox() {
  lightboxOpen.value = false;
}
function prevImage() {
  currentIndex.value =
    (currentIndex.value + images.value.length - 1) % images.value.length;
}
function nextImage() {
  currentIndex.value = (currentIndex.value + 1) % images.value.length;
}
function onImageLoad(e: Event) {
  const img = e.target as HTMLImageElement;
  const card = img.closest(".card");
  card?.classList.add("loaded");
}
const uploadModalOpen = ref(false);
const nickname = ref("");
const fileInput = ref<HTMLInputElement>();
const selectedFiles = ref<File[]>([]);
function getTodayKey() {
  return `uploaded_${new Date().toISOString().slice(0, 10)}`;
}
const uploadedToday = ref<number>(
  Number(localStorage.getItem(getTodayKey()) || 0)
);
const remaining = computed(() => Math.max(27 - uploadedToday.value, 0));
const canSubmit = computed(() => {
  return (
    nickname.value.trim().length > 0 &&
    selectedFiles.value.length > 0 &&
    selectedFiles.value.length <= remaining.value
  );
});
function clearOldUploadRecords() {
  const today = new Date();
  const storage = window.localStorage;
  for (const key of Object.keys(storage)) {
    if (!key.startsWith("uploaded_")) continue;
    const dateStr = key.slice("uploaded_".length);
    const recordDate = new Date(dateStr);
    if (isNaN(recordDate.getTime())) continue;
    const diffMs = today.getTime() - recordDate.getTime();
    const diffDays = diffMs / (1000 * 60 * 60 * 24);
    if (diffDays > 2) {
      storage.removeItem(key);
    }
  }
}
function openUploadModal() {
  clearOldUploadRecords();
  nickname.value = "";
  selectedFiles.value = [];
  if (fileInput.value) fileInput.value.value = "";
  uploadedToday.value = Number(localStorage.getItem(getTodayKey()) || 0);
  uploadModalOpen.value = true;
}
function closeUploadModal() {
  uploadModalOpen.value = false;
}
function handleFileSelect(e: Event) {
  const files = Array.from((e.target as HTMLInputElement).files || []);
  if (!files) return;
  const validFiles: File[] = [];
  for (const file of files) {
    if (file.size > 20 * 1024 * 1024) {
      alert(`文件太大：${file.name}，请控制在 20MB 内`);
      continue;
    }
    validFiles.push(file);
  }
  if (validFiles.length === 0) return;
  if (validFiles.length > remaining.value) {
    alert(
      `今天最多还能上传 ${remaining.value} 张，已为你截取前 ${remaining.value} 张`
    );
    selectedFiles.value = files.slice(0, remaining.value);
  } else {
    selectedFiles.value = files;
  }
}
const isUploading = ref(false);
async function submitUpload() {
  if (!canSubmit.value) return;
  isUploading.value = true;
  try {
    const res = await uploadImages(
      selectedFiles.value,
      nickname.value.trim(),
      "ams"
    );
    const uploadedCount = res.data.length;
    uploadedToday.value += uploadedCount;
    localStorage.setItem(getTodayKey(), String(uploadedToday.value));
    alert(`成功上传 ${uploadedCount} 张图片`);
    closeUploadModal();
  } catch (err: any) {
    console.error(err);
    alert(err.message || "上传失败");
  } finally {
    isUploading.value = false;
  }
}
interface Chibi {
  src: string;
  top: number;
  left: number;
}
const chibiList = ref<Chibi[]>([]);
let sentinelObserver: IntersectionObserver;
onMounted(async () => {
  await fetchRanking();
  await loadNextPage();
  observeNewCards(0);
  sentinelObserver = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) debouncedLoad();
    },
    { rootMargin: "0px", threshold: 0.1 }
  );
  if (sentinel.value) {
    sentinelObserver.observe(sentinel.value);
  }
  const total = 8;
  let pickCount = 3;
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const isMobile = window.innerWidth <= 768;
  const imgWidth = 100;
  const imgHeight = 100;
  function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }
  if (isMobile) {
    pickCount = 1;
  }
  const nums = shuffle(Array.from({ length: total }, (_, k) => k + 1));
  const picks = nums.slice(0, pickCount);
  chibiList.value = [];
  picks.forEach((i) => {
    chibiList.value.push({
      src: `/QImages/1 (${i}).GIF`,
      left: Math.random() * (vw - imgWidth),
      top: Math.random() * (vh - imgHeight),
    });
  });
  await nextTick();
  const imgs = document.querySelectorAll<HTMLImageElement>(".chibi-img");
  imgs.forEach((img, index) => {
    const padding = 200;
    gsap.fromTo(
      img,
      { opacity: 0, scale: 0.5 },
      {
        opacity: 1,
        scale: 1,
        duration: 0.8,
        ease: "back.out(2)",
        delay: 0.2 * index,
      }
    );
    img.addEventListener("mouseenter", () => {
      gsap.killTweensOf(img);
      gsap.to(img, {
        x: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        y: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        duration: 1.2,
        ease: "back.out(2)",
        onComplete: () => {
          animate(img);
        },
      });
    });
    const animate = (img: HTMLImageElement) => {
      let { x, y } = img.getBoundingClientRect();
      let deltaX = (Math.random() - 0.5) * 200;
      let deltaY = (Math.random() - 0.5) * 200;
      let nextX = x + deltaX;
      let nextY = y + deltaY;
      if (nextX < padding) deltaX = padding - x;
      if (nextX + img.width > window.innerWidth - padding)
        deltaX = window.innerWidth - padding - (x + img.width);
      if (nextY < padding) deltaY = padding - y;
      if (nextY + img.height > window.innerHeight - padding)
        deltaY = window.innerHeight - padding - (y + img.height);
      gsap.to(img, {
        x: `+=${deltaX.toFixed(0)}`,
        y: `+=${deltaY.toFixed(0)}`,
        rotation: `+=${((Math.random() - 0.5) * 60).toFixed(0)}`,
        duration: 2 + Math.random() * 2,
        ease: "power1.inOut",
        onComplete: () => animate(img),
      });
    };
    animate(img);
  });
});
onBeforeUnmount(() => {
  observerCard.disconnect();
  sentinelObserver.disconnect();
});
</script>

<style lang="scss" scoped>
/* ===== 基础动画 (新增更多动效) ===== */
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
/* 新增：数据粒子漂浮 */
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
/* 新增：卡片扫描线 */
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
/* 新增：核心能量脉冲 */
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

/* ===== 全局样式 ===== */
.gallery-container {
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
  background: var(--bg-deep);
  color: var(--white-glow);
  min-height: 100vh;
  padding-bottom: 60px;
  padding-top: 20px;
  position: relative;
  overflow-x: hidden;
  font-family: "JetBrains Mono", "Fira Code", "PingFang SC", monospace;
}

/* 故障背景层 - 增强 */
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
    background-size: 40px 40px; /* 网格更密 */
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

/* 新增：漂浮数据粒子层 */
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

/* 新增：核心能量状态装饰 */
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

/* ===== 上传按钮 ===== */
.upload-btn {
  position: fixed;
  bottom: 64px;
  left: 24px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 12px 24px;
  font-size: 1rem;
  z-index: 10;

  border: none;
  color: #1a1028;
  font-weight: bold;
  letter-spacing: 1px;
  background: linear-gradient(135deg, var(--pink-core), var(--pink-light));
  border-radius: 30px;
  box-shadow: 0 0 20px var(--heart-glow), 0 0 40px var(--blue-glitch),
    inset 0 0 10px var(--white-glow);
  transition: all 0.3s ease;
  text-transform: uppercase;
  backdrop-filter: blur(5px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  &::before {
    content: "❤️";
    font-size: 1.2rem;
    filter: drop-shadow(0 0 5px white);
  }
  &:hover {
    transform: translateY(-5px) scale(1.05);
    box-shadow: 0 0 30px var(--pink-core), 0 0 60px var(--blue-glow),
      inset 0 0 15px white;
    animation: glitch 0.3s ease, cursorAnimation_link 1s infinite step-start;
  }
  &:active {
    transform: translateY(-2px);
  }
}

/* ===== 排序按钮 ===== */
.sort-controls {
  margin: 16px 0;
  position: relative;
  z-index: 2;
}
.sort-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 28px;
  background: rgba(26, 16, 40, 0.7);
  backdrop-filter: blur(10px);
  border: 1px solid var(--pink-core);
  border-radius: 40px;
  color: var(--white-glow);
  font-family: inherit;
  font-size: 1rem;

  position: relative;
  overflow: hidden;
  box-shadow: 0 0 15px var(--blue-glitch);
  transition: all 0.3s;
  .btn-glitch-text {
    position: relative;
  }
  .btn-subtext {
    font-size: 0.85rem;
    opacity: 0.8;
    color: var(--blue-glow);
  }
  &:hover {
    border-color: var(--blue-glitch);
    box-shadow: 0 0 25px var(--pink-core), inset 0 0 10px var(--pink-light);
    animation: glitch 0.2s infinite, cursorAnimation_link 1s infinite step-start;
  }
  &::after {
    content: "";
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: linear-gradient(
      to bottom right,
      transparent,
      transparent,
      rgba(255, 255, 255, 0.1),
      transparent
    );
    transform: rotate(30deg);
    animation: shimmer 3s infinite;
  }
}
@keyframes shimmer {
  0% {
    transform: translateX(-100%) rotate(30deg);
  }
  100% {
    transform: translateX(100%) rotate(30deg);
  }
}

/* ===== 画廊网格 ===== */
.gallery {
  padding: 80px 20px;
  max-width: 1200px;
  margin: 0 auto;
  position: relative;
  z-index: 2;
}
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 24px;
}

/* 卡片 - 增强全息感 */
.card {
  perspective: 1000px;
  opacity: 0;
  transform: translateY(20px);
  position: relative;
  &.visible {
    animation: fadeInUp 0.6s ease forwards;
  }
  .card-inner {
    position: relative;
    border-radius: 16px;
    overflow: hidden;
    background: rgba(26, 16, 40, 0.6);
    backdrop-filter: blur(4px);
    border: 2px solid transparent;
    background-clip: padding-box;
    transition: all 0.4s;
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
      border-radius: 18px;
      z-index: -1;
      animation: borderRotate 4s linear infinite;
      opacity: 0;
      transition: opacity 0.3s;
    }
    &:hover::before {
      opacity: 1;
    }
    &:hover {
      transform: rotateY(6deg) rotateX(3deg) scale(1.02);
      .hologram-scan {
        animation: hologram-scan 1.5s linear infinite;
      }
      img {
        filter: brightness(1.1) saturate(1.2);
      }
    }
    img {
      width: 100%;
      aspect-ratio: 1 / 1.2;
      object-fit: cover;
      display: block;
      filter: blur(10px) brightness(0.8);
      opacity: 0.9;
      transition: filter 0.6s, opacity 0.6s, transform 0.4s;
    }
  }
  &.loaded .card-inner img {
    filter: blur(0) brightness(1);
    opacity: 1;
  }
}

/* 故障边框 (增强) */
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

/* 新增：全息扫描线 (hover时出现) */
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

/* 覆盖层 */
.overlay {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 15px 0;
  background: linear-gradient(transparent, rgba(26, 16, 40, 0.9));
  text-align: center;
  opacity: 0;
  transition: opacity 0.4s;
  span {
    color: var(--white-glow);
    font-family: "Fira Code", monospace;
    font-size: 1rem;
    background: rgba(0, 0, 0, 0.5);
    padding: 4px 12px;
    border-radius: 20px;
    border: 1px solid var(--pink-light);
    box-shadow: 0 0 10px var(--blue-glitch);
    letter-spacing: 2px;
    &::after {
      content: "_";
      animation: blink 1s infinite;
    }
  }
}
.card-inner:hover .overlay {
  opacity: 1;
}

/* ===== 点赞按钮 ===== */
.like-btn {
  position: absolute;
  bottom: 12px;
  right: 12px;
  background: rgba(26, 16, 40, 0.7);
  backdrop-filter: blur(5px);
  border: 1px solid var(--pink-core);
  border-radius: 30px;
  animation: cursorAnimation_link 1s infinite step-start;
  z-index: 3;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  transition: all 0.3s;
  .heart {
    width: 24px;
    height: 24px;
    background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="%23ff8cb0" stroke-width="2"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>')
      no-repeat center;
    background-size: contain;
    transition: all 0.3s;
    filter: drop-shadow(0 0 5px var(--pink-core));
  }
  .liked {
    background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%23ff8cb0" stroke="%23ff8cb0"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>')
      no-repeat center;
    background-size: contain;
    animation: pop 0.4s, pulse-glow 2s infinite;
    filter: drop-shadow(0 0 10px var(--heart-glow))
      drop-shadow(0 0 20px var(--blue-glitch));
  }
  .like-count {
    font-size: 1rem;
    color: var(--white-glow);
    font-weight: bold;
    text-shadow: 0 0 8px var(--pink-core);
  }
  &:hover {
    transform: scale(1.15);
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

/* ===== Lightbox ===== */
.lightbox {
  position: fixed;
  inset: 0;
  background: rgba(26, 16, 40, 0.95);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  img {
    max-width: 85%;
    max-height: 85%;
    border-radius: 8px;
    border: 3px solid var(--pink-core);
    box-shadow: 0 0 50px var(--blue-glitch), 0 0 100px var(--pink-core);
    animation: fadeInUp 0.4s;
  }
  .close,
  .prev,
  .next {
    position: absolute;
    color: var(--white-glow);
    font-size: 3rem;
    animation: cursorAnimation_link 1s infinite step-start;
    width: 60px;
    height: 60px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(26, 16, 40, 0.7);
    border: 1px solid var(--pink-core);
    border-radius: 50%;
    transition: all 0.3s;
    &:hover {
      background: var(--pink-core);
      color: #1a1028;
      box-shadow: 0 0 30px var(--blue-glitch);
    }
  }
  .close {
    top: 20px;
    right: 20px;
  }
  .prev {
    left: 20px;
    top: 50%;
    transform: translateY(-50%);
  }
  .next {
    right: 20px;
    top: 50%;
    transform: translateY(-50%);
  }
}

/* ===== 排行榜面板 (增强动画) ===== */
.ranking-panel {
  width: 260px;
  padding: 16px;
  position: fixed;
  top: 64px;
  right: 12px;
  z-index: 100;
  background: rgba(26, 16, 40, 0.8);
  backdrop-filter: blur(12px);
  border: 1px solid var(--pink-core);
  border-radius: 16px;
  box-shadow: 0 0 30px var(--blue-glitch),
    inset 0 0 20px rgba(255, 140, 176, 0.1);
  color: var(--white-glow);
  font-family: "Fira Code", monospace;
  transition: transform 0.3s, box-shadow 0.3s;
  &:hover {
    box-shadow: 0 0 40px var(--pink-core), 0 0 60px var(--blue-glitch);
  }
  .panel-header {
    display: flex;
    align-items: center;
    gap: 8px;
    animation: cursorAnimation_link 1s infinite step-start;
    padding-bottom: 8px;
    border-bottom: 1px dashed var(--pink-light);
    .ranking-title {
      margin: 0;
      font-size: 1.1rem;
      font-weight: 400;
      color: var(--pink-core);
      text-transform: uppercase;
      letter-spacing: 2px;
      text-shadow: 0 0 8px var(--pink-core);
    }
    .stats-badge {
      margin-left: auto;
      font-size: 0.85rem;
      background: rgba(110, 212, 255, 0.2);
      padding: 2px 8px;
      border-radius: 12px;
      border: 1px solid var(--blue-glitch);
    }
    .toggle-icon {
      font-size: 1.2rem;
      color: var(--blue-glow);
      transition: transform 0.3s;
    }
    &:hover .toggle-icon {
      transform: scale(1.2);
    }
  }
  .ranking-list {
    list-style: none;
    padding: 0;
    margin: 12px 0 0;
    max-height: 55vh;
    overflow-y: auto;
    &::-webkit-scrollbar {
      width: 4px;
    }
    &::-webkit-scrollbar-thumb {
      background: var(--pink-core);
      border-radius: 4px;
    }
  }
  .ranking-item {
    display: flex;
    align-items: center;
    padding: 8px 10px;
    margin-bottom: 8px;
    background: rgba(255, 255, 255, 0.03);
    border-left: 3px solid transparent;
    transition: all 0.3s;
    &:hover {
      background: rgba(255, 140, 176, 0.1);
      border-left-color: var(--blue-glitch);
      transform: translateX(-5px);
    }
    .rank {
      width: 30px;
      font-weight: bold;
      color: var(--pink-core);
    }
    .name {
      flex: 1;
      font-size: 0.95rem;
      color: var(--white-glow);
    }
    .count {
      font-size: 0.9rem;
      color: var(--blue-glow);
      font-weight: bold;
    }
    &.rank-1 {
      background: linear-gradient(90deg, rgba(255, 140, 176, 0.2), transparent);
      border-left-color: var(--pink-core);
      .rank,
      .name,
      .count {
        text-shadow: 0 0 8px var(--pink-core);
      }
    }
    &.rank-2 {
      border-left-color: var(--blue-glitch);
    }
    &.rank-3 {
      border-left-color: var(--purple-mid);
    }
  }
}

/* ===== 上传弹窗 ===== */
.upload-modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(26, 16, 40, 0.9);
  backdrop-filter: blur(8px);
}
.upload-modal {
  width: 720px;
  max-width: calc(100% - 40px);
  padding: 36px;
  background: rgba(26, 16, 40, 0.95);
  border: 2px solid var(--pink-core);
  border-radius: 20px;
  box-shadow: 0 0 60px var(--blue-glitch),
    inset 0 0 30px rgba(255, 140, 176, 0.2);
  color: var(--white-glow);
  font-family: "Fira Code", monospace;
  h3 {
    margin: 0 0 20px;
    font-size: 1.8rem;
    color: var(--pink-core);
    text-shadow: 0 0 15px var(--pink-core);
    letter-spacing: 4px;
    &::after {
      content: "_";
      animation: blink 1s infinite;
    }
  }
  .tip-container {
    margin: 20px 0;
    padding: 16px;
    background: rgba(0, 0, 0, 0.3);
    border: 1px dashed var(--blue-glitch);
    border-radius: 12px;
    .tips-list {
      list-style: none;
      padding: 0;
      li {
        margin-bottom: 8px;
        padding-left: 20px;
        position: relative;
        &::before {
          content: ">";
          position: absolute;
          left: 0;
          color: var(--pink-core);
        }
      }
    }
  }
  .stats {
    margin: 16px 0;
    padding: 12px;
    background: rgba(110, 212, 255, 0.1);
    border-radius: 8px;
    text-align: center;
    strong {
      color: var(--pink-core);
      font-size: 1.2rem;
    }
  }
  label {
    display: block;
    margin-bottom: 20px;
    .label-icon {
      color: var(--blue-glitch);
      margin-right: 8px;
    }
    input {
      width: 100%;
      margin-top: 6px;
      padding: 12px;
      background: rgba(0, 0, 0, 0.3);
      border: 1px solid var(--pink-light);
      border-radius: 8px;
      color: var(--white-glow);
      font-family: inherit;
      &:focus {
        outline: none;
        border-color: var(--blue-glitch);
        box-shadow: 0 0 20px var(--blue-glitch);
      }
    }
  }
  .modal-actions {
    display: flex;
    justify-content: flex-end;
    gap: 16px;
    margin-top: 30px;
    button {
      padding: 12px 30px;
      border: none;
      border-radius: 30px;
      font-weight: bold;
      animation: cursorAnimation_link 1s infinite step-start;
      transition: all 0.3s;
      background: linear-gradient(135deg, var(--pink-core), var(--pink-light));
      color: #1a1028;
      border: 1px solid transparent;
      &:hover:not(:disabled) {
        transform: translateY(-3px);
        box-shadow: 0 0 30px var(--pink-core), 0 0 60px var(--blue-glitch);
      }
      &:disabled {
        opacity: 0.4;
        animation: cursorAnimation_disabled 1s infinite step-start;
      }
      &.cancel {
        background: transparent;
        border: 1px solid var(--blue-glitch);
        color: var(--blue-glow);
        &:hover {
          background: rgba(110, 212, 255, 0.1);
          box-shadow: 0 0 20px var(--blue-glitch);
        }
      }
    }
  }
}

/* ===== 浮动小宠物 ===== */
.floating-chibis {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 5;
}
.chibi-img {
  position: absolute;
  width: 80px;
  user-select: none;
  pointer-events: auto;
  filter: drop-shadow(0 0 10px var(--pink-glow))
    drop-shadow(0 0 20px var(--blue-glitch));
  opacity: 0.8;
  transition: filter 0.3s;
  &:hover {
    filter: drop-shadow(0 0 20px var(--pink-core))
      drop-shadow(0 0 40px var(--blue-glow));
  }
}

/* 加载状态 */
.loading,
.finished {
  text-align: center;
  padding: 20px;
  color: var(--blue-glow);
  font-family: "Fira Code", monospace;
  .blink {
    animation: blink 1s infinite;
  }
}

/* 工具类 */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
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

/* 响应式 */
@media (max-width: 980px) {
  .gallery {
    padding: 40px 14px;
  }
  .ranking-panel {
    width: 220px;
  }
}
@media (max-width: 768px) {
  .upload-btn {
    bottom: 30px;
  }
  .core-status {
    display: none;
  } /* 移动端隐藏装饰 */
}
</style>
