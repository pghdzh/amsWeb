<template>
  <div class="aimis-trap-game">
    <!-- 故障光效背景层 -->
    <div class="glitch-bg"></div>
    <div class="data-particles"></div>
    <div class="scan-line"></div>



    <!-- 游戏主布局 -->
    <div class="game-layout">
      <!-- 左侧：游戏棋盘区 -->
      <div class="game-main">
        <div class="game-header">
          <h1 class="glitch-text" data-text="围住阿漂">围住阿漂</h1>
          <div class="game-stats">
            <div class="stat-item">
              <span class="stat-icon">👣</span>
              <span class="stat-label">步数</span>
              <span class="stat-value">{{ moveCount }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-icon">✨</span>
              <span class="stat-label">路障</span>
              <span class="stat-value">{{ blockCount }}</span>
            </div>
            <div
              class="stat-item current-turn"
              :class="`player-${currentPlayer}`"
            >
              <span class="stat-label">当前</span>
              <span class="stat-value">漂泊者 {{ currentPlayer }}</span>
            </div>
          </div>
        </div>

        <!-- 画布容器 -->
        <div class="canvas-container" ref="canvasContainer">
          <canvas
            ref="boardCanvas"
            class="board-canvas"
            @click="handleCanvasClick"
            @mousemove="handleMouseMove"
            @mouseleave="hoverCell = null"
          ></canvas>
        </div>

        <!-- 底部控制按钮 -->
        <div class="game-controls">
          <button class="control-btn" @click="resetGame">重新开始</button>
          <button class="control-btn" @click="toggleRules">
            {{ showRules ? "隐藏规则" : "游戏规则" }}
          </button>
          <button class="control-btn" @click="openRanking">
            {{ rankingDrawerOpen ? "隐藏排行" : "查看排行" }}
          </button>
        </div>
      </div>

      <!-- 右侧：排行榜面板（可折叠） -->
      <div class="right-panel" :class="{ 'panel-open': rankingDrawerOpen }">
        <div class="panel-header">
          <h3>▌ 星辉排行榜</h3>
          <button
            class="close-btn"
            @click="rankingDrawerOpen = false"
            v-if="rankingDrawerOpen"
          >
            ✕
          </button>
        </div>
        <div class="panel-content">
          <div v-if="rankingLoading" class="panel-loading">
            <div class="small-spinner"></div>
            加载中...
          </div>
          <ul v-else-if="rankingList.length" class="ranking-list">
            <li
              v-for="(item, idx) in rankingList"
              :key="idx"
              class="ranking-item"
              :class="`rank-${idx + 1}`"
            >
              <span class="rank">{{ idx + 1 }}</span>
              <span class="name">{{ item.nickname }}</span>
              <span class="score">{{ item.count }} 步</span>
            </li>
          </ul>
          <div v-else class="ranking-empty">
            <span class="empty-icon">🏆</span>
            <p>暂无记录</p>
          </div>
        </div>
        <div class="panel-footer">
          <span class="tip">✨ 步数越多排名越高</span>
        </div>
      </div>
    </div>

    <!-- 游戏状态弹窗 -->
    <transition name="fade">
      <div
        class="game-overlay"
        v-if="gameState !== 'playing' && gameState !== 'ready'"
      >
        <div class="overlay-content">
          <h2
            v-if="gameState === 'win'"
            class="glitch-text"
            data-text="围住啦！"
          >
            围住啦！
          </h2>
          <h2 v-else-if="gameState === 'lose'">逃走了...</h2>

          <div class="score-result" v-if="gameState === 'win'">
            <div class="result-stats">
              <span>总步数：{{ moveCount }}</span>
              <span>放置路障：{{ blockCount }}</span>
            </div>
            <div class="score-submit" v-if="!scoreSubmitted">
              <input
                v-model="nickname"
                type="text"
                placeholder="你的昵称"
                maxlength="20"
              />
              <button
                class="game-btn"
                @click="submitScore"
                :disabled="submitting || !nickname.trim()"
              >
                {{ submitting ? "提交中..." : "提交分数" }}
              </button>
            </div>
            <div class="submitted-tip" v-else>
              ✅ 已提交 · 步数 {{ moveCount }}
            </div>
          </div>

          <div class="overlay-btns">
            <button class="game-btn" @click="resetGame">再玩一次</button>
          </div>
          <div class="aimis-quote">
            {{ gameState === "win" ? winQuote : loseQuote }}
          </div>
        </div>
      </div>
    </transition>

    <!-- 规则弹窗 -->
    <transition name="fade">
      <div
        class="rules-overlay"
        v-if="showRules"
        @click.self="showRules = false"
      >
        <div class="rules-content">
          <h3>📖 游戏规则</h3>
          <ul>
            <li>
              点击空白六边形，放置<span class="highlight">爱弥斯路障</span>
            </li>
            <li>
              两个漂泊者交替向出口移动，每次你放置一个路障，当前行动的漂泊者移动一步
            </li>
            <li>
              <span class="highlight">必须同时围住两个漂泊者才能获胜</span>
              （即他们均无法到达边界）
            </li>
            <li>
              如果某个漂泊者周围六格完全被堵，它会显示锁图标，但仍需围住另一个
            </li>
            <li>漂泊者跑到边界则游戏失败</li>
            <li>
              <span class="highlight">步数越多排名越高</span
              >——用更少的冲动，更多的策略
            </li>
            <li>
              阿漂并不是一直按照最优策略行动,<span class="highlight"
                >有一定几率随机走</span
              >
            </li>
          </ul>
          <div class="rule-tip">💡 爱弥斯说：“我会慢慢把你们围起来~”</div>
          <button class="close-btn" @click="showRules = false">知道了</button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, nextTick } from "vue";
import {
  getRankingList,
  addRankingItem,
  updateRankingItem,
} from "@/api/modules/ranking";
import aimisAvatar from "@/assets/avatar/ams.png";
import womanAvatar from "@/assets/avatar/woman.png";
import manAvatar from "@/assets/avatar/man.png";

// ========== 配置 ==========
const CHARACTER_KEY = "aimis_trap";
const BOARD_RADIUS = 5; // 棋盘半径
const CELL_SIZE = 64; // 六边形边长（像素）
const INITIAL_BLOCKS = 6; // 初始路障数量
const ANIMATION_DURATION = 200; // 移动动画时长（毫秒）

const DIRECTIONS = [
  { dx: 1, dy: 0 }, // 右
  { dx: 0, dy: 1 }, // 右下
  { dx: -1, dy: 1 }, // 左下
  { dx: -1, dy: 0 }, // 左
  { dx: 0, dy: -1 }, // 左上
  { dx: 1, dy: -1 }, // 右上
];

// ========== 游戏状态 ==========
type GameState = "ready" | "playing" | "win" | "lose";
const gameState = ref<GameState>("ready");
const coreEnergy = ref(98);
const moveCount = ref(0);
const blockCount = ref(0);
const showRules = ref(false);
const rankingDrawerOpen = ref(false);
const isMobile = ref(window.innerWidth <= 768);

// 无法到达边界（用于胜利）
const player1NoExit = ref(false);
const player2NoExit = ref(false);
// 被包围（周围无空位，用于锁图标）
const player1Surrounded = ref(false);
const player2Surrounded = ref(false);

// 头像
const player1Avatar = ref(Math.random() > 0.5 ? womanAvatar : manAvatar);
const player2Avatar = ref(
  player1Avatar.value === womanAvatar ? manAvatar : womanAvatar
);
const aimisAvatarImg = ref<HTMLImageElement>(new Image());
aimisAvatarImg.value.src = aimisAvatar;

// 棋盘数据
interface Cell {
  q: number;
  r: number;
  x: number; // 物理坐标（基于原点）
  y: number;
  type: "empty" | "block";
  player: 0 | 1 | 2; // 0 无玩家，1 玩家1，2 玩家2
  trapped: boolean; // 是否被困（用于锁图标）
}

const board = ref<Cell[]>([]);
const player1Pos = ref({ q: 0, r: 0 });
const player2Pos = ref({ q: 0, r: 0 });
const currentPlayer = ref<1 | 2>(1);

// 动画相关
let animFrame: number | null = null;
let animStartTime: number | null = null;
let animPlayer: 1 | 2 | null = null;
let animFrom: { q: number; r: number } | null = null;
let animTo: { q: number; r: number } | null = null;
let animProgress = 0; // 0-1

// 画布相关
const canvasContainer = ref<HTMLElement | null>(null);
const boardCanvas = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D | null = null;
let hoverCell: Cell | null = null;

// 排行榜
const rankingList = ref<any[]>([]);
const rankingLoading = ref(false);
const nickname = ref("");
const submitting = ref(false);
const scoreSubmitted = ref(false);

// 爱弥斯语录
const winQuotes = [
  "哼哼，两个都被我围住了~",
  "完美包围！",
  "星辉路障，全面封锁",
];
const loseQuotes = ["哎呀，跑掉了...再来一次", "下一次一定围住你们"];
const winQuote = ref(winQuotes[0]);
const loseQuote = ref(loseQuotes[0]);

// ========== 工具函数 ==========
const getCell = (q: number, r: number) =>
  board.value.find((c) => c.q === q && c.r === r);

const isBorder = (q: number, r: number) =>
  Math.abs(q) === BOARD_RADIUS ||
  Math.abs(r) === BOARD_RADIUS ||
  Math.abs(q + r) === BOARD_RADIUS;

// 判断玩家是否能到达边界（BFS）
const canReachBorder = (q: number, r: number, playerId: 1 | 2) => {
  const opponentPos = playerId === 1 ? player2Pos.value : player1Pos.value;
  const visited = new Set<string>();
  const queue: { q: number; r: number }[] = [{ q, r }];
  visited.add(`${q},${r}`);

  while (queue.length > 0) {
    const current = queue.shift()!;
    if (isBorder(current.q, current.r)) return true;

    for (const dir of DIRECTIONS) {
      const nq = current.q + dir.dx;
      const nr = current.r + dir.dy;
      const neighbor = getCell(nq, nr);
      if (neighbor) {
        const isOpponent = nq === opponentPos.q && nr === opponentPos.r;
        if (
          neighbor.type === "empty" &&
          !isOpponent &&
          !visited.has(`${nq},${nr}`)
        ) {
          visited.add(`${nq},${nr}`);
          queue.push({ q: nq, r: nr });
        }
      }
    }
  }
  return false;
};

// 判断玩家是否被包围（周围六格无空格，对方玩家视为障碍）
const isSurrounded = (q: number, r: number, playerId: 1 | 2) => {
  const opponentPos = playerId === 1 ? player2Pos.value : player1Pos.value;
  for (const dir of DIRECTIONS) {
    const nq = q + dir.dx;
    const nr = r + dir.dy;
    const neighbor = getCell(nq, nr);
    if (neighbor) {
      if (neighbor.type === "empty") {
        if (!(nq === opponentPos.q && nr === opponentPos.r)) {
          return false; // 还有空格可走
        }
      }
    }
  }
  return true; // 所有方向都被堵
};

// 更新无法到达边界的状态（胜利条件）
const updateNoExitStatus = () => {
  player1NoExit.value = !canReachBorder(
    player1Pos.value.q,
    player1Pos.value.r,
    1
  );
  player2NoExit.value = !canReachBorder(
    player2Pos.value.q,
    player2Pos.value.r,
    2
  );
};

// 更新被包围状态（锁图标）
const updateSurroundedStatus = () => {
  player1Surrounded.value = isSurrounded(
    player1Pos.value.q,
    player1Pos.value.r,
    1
  );
  player2Surrounded.value = isSurrounded(
    player2Pos.value.q,
    player2Pos.value.r,
    2
  );

  board.value.forEach((cell) => {
    if (cell.player === 1) cell.trapped = player1Surrounded.value;
    else if (cell.player === 2) cell.trapped = player2Surrounded.value;
    else cell.trapped = false;
  });
};

// 检查游戏状态
const checkGameStatus = () => {
  // 胜利：双方均无法到达边界
  if (player1NoExit.value && player2NoExit.value) {
    gameState.value = "win";
    return;
  }
  // 失败：有玩家跑出边界
  if (
    isBorder(player1Pos.value.q, player1Pos.value.r) ||
    isBorder(player2Pos.value.q, player2Pos.value.r)
  ) {
    gameState.value = "lose";
    return;
  }
};

// BFS找最近出口（避开路障和对方玩家），加入随机因子
const findNextMove = (fromQ: number, fromR: number, playerId: 1 | 2) => {
  if (isBorder(fromQ, fromR)) return null;
  const opponentPos = playerId === 1 ? player2Pos.value : player1Pos.value;

  // 收集所有可走的直接邻居
  const validNeighbors: { q: number; r: number }[] = [];
  for (const dir of DIRECTIONS) {
    const nq = fromQ + dir.dx;
    const nr = fromR + dir.dy;
    const neighbor = getCell(nq, nr);
    if (neighbor && neighbor.type === "empty") {
      const isOpponent = nq === opponentPos.q && nr === opponentPos.r;
      if (!isOpponent) {
        validNeighbors.push({ q: nq, r: nr });
      }
    }
  }

  // 如果没有任何可走邻居，返回 null（被包围）
  if (validNeighbors.length === 0) {
    return null;
  }

  // 随机概率：20% 直接随机选一个邻居（降低AI强度）
  if (validNeighbors.length > 1 && Math.random() < 0.05) {
    return validNeighbors[Math.floor(Math.random() * validNeighbors.length)];
  }

  // 否则 BFS 找最近出口
  const queue: { q: number; r: number; path: { q: number; r: number }[] }[] =
    [];
  const visited = new Set<string>();
  queue.push({ q: fromQ, r: fromR, path: [] });
  visited.add(`${fromQ},${fromR}`);

  while (queue.length > 0) {
    const current = queue.shift()!;
    if (isBorder(current.q, current.r)) {
      return current.path[0] || null;
    }

    const shuffledDirs = [...DIRECTIONS].sort(() => Math.random() - 0.5);
    for (const dir of shuffledDirs) {
      const nq = current.q + dir.dx;
      const nr = current.r + dir.dy;
      const neighbor = getCell(nq, nr);
      if (neighbor) {
        const isOpponent = nq === opponentPos.q && nr === opponentPos.r;
        if (
          neighbor.type === "empty" &&
          !isOpponent &&
          !visited.has(`${nq},${nr}`)
        ) {
          visited.add(`${nq},${nr}`);
          queue.push({
            q: nq,
            r: nr,
            path: [...current.path, { q: nq, r: nr }],
          });
        }
      }
    }
  }

  // BFS 找不到出口，但还有邻居，随机选择一个（比如被困在内部循环）
  if (validNeighbors.length > 0) {
    return validNeighbors[Math.floor(Math.random() * validNeighbors.length)];
  }
  return null;
};

// 移动玩家（启动动画）
const movePlayer = (playerId: 1 | 2) => {
  // 如果该玩家已被包围，不移动，直接切换（但状态已在动画结束后更新）
  if (
    (playerId === 1 && player1Surrounded.value) ||
    (playerId === 2 && player2Surrounded.value)
  ) {
    if (gameState.value === "playing") {
      currentPlayer.value = currentPlayer.value === 1 ? 2 : 1;
    }
    return;
  }

  const pos = playerId === 1 ? player1Pos.value : player2Pos.value;
  const nextMove = findNextMove(pos.q, pos.r, playerId);

  if (!nextMove) {
    // 无路可走（被完全包围）
    updateNoExitStatus();
    updateSurroundedStatus();
    if (player1NoExit.value && player2NoExit.value) {
      gameState.value = "win";
      return;
    }
    // 切换玩家
    if (gameState.value === "playing") {
      currentPlayer.value = currentPlayer.value === 1 ? 2 : 1;
    }
    return;
  }

  // 有路可走，启动动画
  startAnimation(playerId, pos, nextMove);
};

// ========== 动画系统 ==========
const startAnimation = (
  player: 1 | 2,
  from: { q: number; r: number },
  to: { q: number; r: number }
) => {
  if (animFrame) cancelAnimationFrame(animFrame);
  animPlayer = player;
  animFrom = from;
  animTo = to;
  animStartTime = performance.now();
  animProgress = 0;

  const animate = (now: number) => {
    if (!animStartTime) return;
    const elapsed = now - animStartTime;
    animProgress = Math.min(elapsed / ANIMATION_DURATION, 1);
    drawBoard(); // 重绘，动画位置会插值
    if (animProgress < 1) {
      animFrame = requestAnimationFrame(animate);
    } else {
      // 动画完成，更新实际位置
      if (animFrom && animTo && animPlayer) {
        const fromCell = getCell(animFrom.q, animFrom.r);
        const toCell = getCell(animTo.q, animTo.r);
        if (fromCell && toCell) {
          fromCell.player = 0;
          toCell.player = animPlayer;
          if (animPlayer === 1) player1Pos.value = { q: animTo.q, r: animTo.r };
          else player2Pos.value = { q: animTo.q, r: animTo.r };
        }
      }
      animPlayer = null;
      animFrom = null;
      animTo = null;

      // 更新状态
      updateNoExitStatus();
      updateSurroundedStatus();
      checkGameStatus();

      // 如果游戏仍在进行，切换当前玩家
      if (gameState.value === "playing") {
        currentPlayer.value = currentPlayer.value === 1 ? 2 : 1;
      }

      drawBoard();
      animFrame = null;
    }
  };

  animFrame = requestAnimationFrame(animate);
};

// ========== 棋盘初始化 ==========
const initBoard = () => {
  const cells: Cell[] = [];
  for (let q = -BOARD_RADIUS; q <= BOARD_RADIUS; q++) {
    const r1 = Math.max(-BOARD_RADIUS, -q - BOARD_RADIUS);
    const r2 = Math.min(BOARD_RADIUS, -q + BOARD_RADIUS);
    for (let r = r1; r <= r2; r++) {
      const x = q * 1.5 * CELL_SIZE;
      const y = (r * Math.sqrt(3) + q * (Math.sqrt(3) / 2)) * CELL_SIZE;
      cells.push({ q, r, x, y, type: "empty", player: 0, trapped: false });
    }
  }
  board.value = cells;

  // 放置两个玩家（不靠边且不重叠）
  const placePlayer = (playerId: 1 | 2) => {
    for (let attempt = 0; attempt < 200; attempt++) {
      const randIdx = Math.floor(Math.random() * cells.length);
      const cell = cells[randIdx];
      if (cell.type !== "empty" || cell.player !== 0) continue;
      if (isNearBorder(cell.q, cell.r, 2)) continue;
      cell.player = playerId;
      return { q: cell.q, r: cell.r };
    }
    // 保底：中心区域
    for (let q = -1; q <= 1; q++) {
      for (let r = -1; r <= 1; r++) {
        if (Math.abs(q + r) <= 1) {
          const cell = cells.find((c) => c.q === q && c.r === r);
          if (cell && cell.type === "empty" && cell.player === 0) {
            cell.player = playerId;
            return { q, r };
          }
        }
      }
    }
    throw new Error("无法放置玩家");
  };

  player1Pos.value = placePlayer(1);
  player2Pos.value = placePlayer(2);
  currentPlayer.value = 1;

  // 初始路障
  let blocksPlaced = 0;
  while (blocksPlaced < INITIAL_BLOCKS) {
    const randIdx = Math.floor(Math.random() * cells.length);
    const cell = cells[randIdx];
    if (cell.type === "empty" && cell.player === 0) {
      cell.type = "block";
      blocksPlaced++;
    }
  }

  moveCount.value = 0;
  blockCount.value = 0;
  scoreSubmitted.value = false;
  gameState.value = "playing";
  resizeCanvas();
  updateNoExitStatus();
  updateSurroundedStatus();
};

const isNearBorder = (q: number, r: number, threshold: number) =>
  Math.abs(q) >= BOARD_RADIUS - threshold ||
  Math.abs(r) >= BOARD_RADIUS - threshold ||
  Math.abs(q + r) >= BOARD_RADIUS - threshold;

// ========== 画布自适应 ==========
const resizeCanvas = () => {
  const canvas = boardCanvas.value;
  const container = canvasContainer.value;
  if (!canvas || !container || board.value.length === 0) return;

  let minX = Infinity,
    maxX = -Infinity,
    minY = Infinity,
    maxY = -Infinity;
  board.value.forEach((cell) => {
    minX = Math.min(minX, cell.x);
    maxX = Math.max(maxX, cell.x);
    minY = Math.min(minY, cell.y);
    maxY = Math.max(maxY, cell.y);
  });
  const padding = CELL_SIZE * 1.5;
  const boardWidth = maxX - minX + 2 * padding;
  const boardHeight = maxY - minY + 2 * padding;

  canvas.width = boardWidth;
  canvas.height = boardHeight;
  canvas.style.width = "100%";
  canvas.style.height = "100%";
  canvas.style.objectFit = "fill"; // 保证画布填满容器

  // 存储偏移量
  (canvas as any)._offsetX = -minX + padding;
  (canvas as any)._offsetY = -minY + padding;
  if (canvasContainer.value) {
    const aspect = boardWidth / boardHeight;
    canvasContainer.value.style.aspectRatio = `${aspect}`;
  }
  drawBoard();
};

// ========== 绘制 ==========
const drawBoard = () => {
  if (!boardCanvas.value || !ctx) return;
  const canvas = boardCanvas.value;
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  const offsetX = (canvas as any)._offsetX || 0;
  const offsetY = (canvas as any)._offsetY || 0;

  board.value.forEach((cell) => {
    const x = cell.x + offsetX;
    const y = cell.y + offsetY;
    const size = CELL_SIZE * 0.7;

    // 绘制六边形背景
    ctx.beginPath();
    for (let i = 0; i < 6; i++) {
      const angle = (i * Math.PI) / 3;
      const px = x + size * Math.cos(angle);
      const py = y + size * Math.sin(angle);
      if (i === 0) ctx.moveTo(px, py);
      else ctx.lineTo(px, py);
    }
    ctx.closePath();

    // 填充背景
    if (cell.type === "block") {
      ctx.fillStyle = "rgba(0,0,0,0.3)";
      ctx.shadowColor = "#ff8cb0";
      ctx.shadowBlur = 15;
    } else if (cell.player !== 0) {
      ctx.fillStyle = "rgba(110,212,255,0.2)";
      ctx.shadowColor = "#6ed4ff";
      ctx.shadowBlur = 20;
    } else {
      ctx.fillStyle = "rgba(255,255,255,0.05)";
      ctx.shadowBlur = 0;
    }
    ctx.fill();

    // 边框
    if (cell.player !== 0) {
      // 如果是当前玩家且未被锁，加金色边框；否则白色
      if (cell.player === currentPlayer.value && !cell.trapped) {
        ctx.strokeStyle = "#ffd700";
        ctx.lineWidth = 4;
        ctx.shadowBlur = 20;
        ctx.shadowColor = "#ffd700";
      } else {
        ctx.strokeStyle = "#ffffff";
        ctx.lineWidth = 2;
        ctx.shadowBlur = 10;
        ctx.shadowColor = "#6ed4ff";
      }
    } else if (cell.type === "block") {
      ctx.strokeStyle = "rgba(255,140,176,0.8)";
      ctx.lineWidth = 2;
      ctx.shadowBlur = 10;
      ctx.shadowColor = "#ff8cb0";
    } else {
      ctx.strokeStyle = "rgba(255,140,176,0.3)";
      ctx.lineWidth = 1;
      ctx.shadowBlur = 0;
    }
    ctx.stroke();

    // 绘制内容
    if (cell.type === "block") {
      if (aimisAvatarImg.value.complete) {
        ctx.drawImage(
          aimisAvatarImg.value,
          x - size / 2,
          y - size / 2,
          size,
          size
        );
      } else {
        ctx.font = `${size / 2}px Arial`;
        ctx.fillStyle = "white";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillText("❤️", x, y);
      }
    } else if (cell.player === 1) {
      const img = player1AvatarImg.value;
      if (img.complete) {
        ctx.drawImage(img, x - size / 2, y - size / 2, size, size);
      } else {
        ctx.font = `${size / 2}px Arial`;
        ctx.fillStyle = "white";
        ctx.fillText("👤", x, y);
      }
    } else if (cell.player === 2) {
      const img = player2AvatarImg.value;
      if (img.complete) {
        ctx.drawImage(img, x - size / 2, y - size / 2, size, size);
      } else {
        ctx.font = `${size / 2}px Arial`;
        ctx.fillStyle = "white";
        ctx.fillText("👤", x, y);
      }
    }

    // 如果玩家被困，绘制锁图标
    if (cell.trapped && cell.player !== 0) {
      ctx.font = `${size / 3}px Arial`;
      ctx.fillStyle = "rgba(255,255,255,0.9)";
      ctx.shadowBlur = 10;
      ctx.shadowColor = "#ff0000";
      ctx.fillText("🔒", x + size / 4, y - size / 4);
    }

    // 出口提示（轻淡）
    if (
      isBorder(cell.q, cell.r) &&
      cell.type === "empty" &&
      cell.player === 0
    ) {
      ctx.font = "12px Arial";
      ctx.fillStyle = "rgba(255,255,255,0.2)";
      ctx.shadowBlur = 0;
      ctx.fillText("出口", x, y - 10);
    }

    // hover 高亮（仅空格）
    if (
      hoverCell &&
      cell.q === hoverCell.q &&
      cell.r === hoverCell.r &&
      cell.type === "empty" &&
      cell.player === 0
    ) {
      ctx.strokeStyle = "white";
      ctx.lineWidth = 4;
      ctx.shadowBlur = 20;
      ctx.shadowColor = "#ff8cb0";
      ctx.stroke();
    }
  });

  // 动画中绘制移动的玩家
  if (
    animPlayer &&
    animFrom &&
    animTo &&
    animProgress > 0 &&
    animProgress < 1
  ) {
    const fromCell = getCell(animFrom.q, animFrom.r);
    const toCell = getCell(animTo.q, animTo.r);
    if (fromCell && toCell) {
      const fromX = fromCell.x + offsetX;
      const fromY = fromCell.y + offsetY;
      const toX = toCell.x + offsetX;
      const toY = toCell.y + offsetY;
      const curX = fromX + (toX - fromX) * animProgress;
      const curY = fromY + (toY - fromY) * animProgress;
      const size = CELL_SIZE * 0.7;
      const img =
        animPlayer === 1 ? player1AvatarImg.value : player2AvatarImg.value;
      if (img.complete) {
        ctx.save();
        ctx.globalAlpha = 0.8;
        ctx.drawImage(img, curX - size / 2, curY - size / 2, size, size);
        ctx.restore();
      }
    }
  }
};

// 预加载图片
const player1AvatarImg = ref<HTMLImageElement>(new Image());
const player2AvatarImg = ref<HTMLImageElement>(new Image());
player1AvatarImg.value.src = player1Avatar.value;
player2AvatarImg.value.src = player2Avatar.value;
aimisAvatarImg.value.onload = () => drawBoard();

// ========== 用户交互 ==========
const getCellFromMouse = (e: MouseEvent): Cell | null => {
  const canvas = boardCanvas.value;
  if (!canvas) return null;
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  const mouseCanvasX = (e.clientX - rect.left) * scaleX;
  const mouseCanvasY = (e.clientY - rect.top) * scaleY;

  const offsetX = (canvas as any)._offsetX || 0;
  const offsetY = (canvas as any)._offsetY || 0;
  const rawX = mouseCanvasX - offsetX;
  const rawY = mouseCanvasY - offsetY;

  let minDist = Infinity;
  let closestCell: Cell | null = null;
  board.value.forEach((cell) => {
    const dx = rawX - cell.x;
    const dy = rawY - cell.y;
    const dist = Math.sqrt(dx * dx + dy * dy);
    if (dist < CELL_SIZE && dist < minDist) {
      minDist = dist;
      closestCell = cell;
    }
  });
  return closestCell;
};

const handleCanvasClick = (e: MouseEvent) => {
  if (gameState.value !== "playing") return;
  if (animFrame) return; // 动画中禁止点击
  const cell = getCellFromMouse(e);
  if (!cell) return;
  if (cell.type !== "empty" || cell.player !== 0) return;

  cell.type = "block";
  blockCount.value++;
  moveCount.value++;
  coreEnergy.value = Math.min(100, coreEnergy.value + 2);

  // 移动当前玩家
  movePlayer(currentPlayer.value);

  // 如果游戏仍在进行，切换玩家（但 movePlayer 或动画结束后会切换，这里防止重复）
  // 实际上 movePlayer 内部已经切换了，但为了安全，我们仍可以在无动画时处理
  // 但动画结束后会切换，所以这里不需要额外处理，只需重绘
  drawBoard();
};

const handleMouseMove = (e: MouseEvent) => {
  hoverCell = getCellFromMouse(e);
  drawBoard();
};

// ========== 排行榜 ==========
const fetchRanking = async () => {
  rankingLoading.value = true;
  try {
    const res = await getRankingList({
      page: 1,
      pageSize: 999,
      character_key: CHARACTER_KEY,
    });
    rankingList.value = res?.success
      ? Array.isArray(res.data)
        ? res.data
        : res.data?.items ?? []
      : [];
  } catch {
    rankingList.value = [];
  } finally {
    rankingLoading.value = false;
  }
};

const submitScore = async () => {
  if (!nickname.value.trim() || submitting.value) return;
  submitting.value = true;
  try {
    // 先获取最新排行榜，确保检查时数据是最新的
    await fetchRanking();
    const existingItem = rankingList.value.find(
      (item) => item.nickname === nickname.value.trim()
    );

    if (existingItem) {
      // 存在同名记录，比较分数
      if (moveCount.value > existingItem.count) {
        // 新分数更高，更新该记录
        const updateRes = await updateRankingItem(existingItem.id, {
          count: moveCount.value,
        });
        if (updateRes?.success) {
          await fetchRanking(); // 刷新排行榜
          scoreSubmitted.value = true;
          rankingDrawerOpen.value = true;
        } else {
          alert("更新失败，请稍后重试");
        }
      } else {
        // 未超越历史最高分
        alert(
          `你的最高记录是 ${existingItem.count} 步，本次 ${moveCount.value} 步未超越`
        );
        // 不提交，直接返回
      }
    } else {
      // 无同名记录，直接添加
      const res = await addRankingItem({
        character_key: CHARACTER_KEY,
        nickname: nickname.value.trim(),
        count: moveCount.value,
      });
      if (res?.success) {
        await fetchRanking();
        scoreSubmitted.value = true;
        rankingDrawerOpen.value = true;
      } else {
        alert("提交失败，请稍后重试");
      }
    }
  } catch (err) {
    console.error("提交分数异常", err);
    alert("提交过程中发生错误");
  } finally {
    submitting.value = false;
  }
};

// ========== 控制 ==========
const resetGame = () => {
  if (animFrame) cancelAnimationFrame(animFrame);
  animPlayer = null;
  animFrom = null;
  animTo = null;
  initBoard();
  winQuote.value = winQuotes[Math.floor(Math.random() * winQuotes.length)];
  loseQuote.value = loseQuotes[Math.floor(Math.random() * loseQuotes.length)];
  gameState.value = "playing";
  hoverCell = null;
};

const toggleRules = () => {
  showRules.value = !showRules.value;
};
const openRanking = () => {
  rankingDrawerOpen.value = !rankingDrawerOpen.value;
  if (rankingDrawerOpen.value) fetchRanking();
};

// ========== 窗口监听 ==========
const handleResize = () => {
  isMobile.value = window.innerWidth <= 768;
  resizeCanvas();
};

// ========== 生命周期 ==========
onMounted(() => {
  ctx = boardCanvas.value?.getContext("2d") || null;
  initBoard();
  fetchRanking();
  window.addEventListener("resize", handleResize);
  player1AvatarImg.value.onload = () => drawBoard();
  player2AvatarImg.value.onload = () => drawBoard();
  nextTick(() => resizeCanvas());
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
  if (animFrame) cancelAnimationFrame(animFrame);
});
</script>

<style scoped lang="scss">
.aimis-trap-game {
  --pink-core: #ff8cb0;
  --blue-glitch: #6ed4ff;
  --bg-deep: linear-gradient(145deg, #1a1025 0%, #23182b 100%);
  --grid-color: rgba(255, 140, 176, 0.1);
  --heart-glow: rgba(255, 140, 176, 0.5);

  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: var(--bg-deep);
  color: white;
  font-family: "Noto Sans SC", sans-serif;

  // 背景层
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

  .glitch-bg,
  .data-particles,
  .scan-line {
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 1;
  }


  .game-layout {
    position: relative;
    width: 100%;
    height: 100%;
    display: flex;
    padding: 80px 20px 20px;
    box-sizing: border-box;
    gap: 20px;
    z-index: 5;
  }

  .game-main {
    flex: 1;
    display: flex;
    flex-direction: column;
    min-width: 0;
    align-items: center;
    justify-content: center;
  }

  .game-header {
    text-align: center;
    margin-bottom: 20px;
    h1 {
      font-size: 2.2rem;
      margin: 0 0 10px;
      color: var(--pink-core);
      text-shadow: 0 0 15px var(--pink-core);
      &.glitch-text {
        animation: glitch 3s infinite;
      }
    }
    .game-stats {
      display: flex;
      gap: 30px;
      justify-content: center;
      .stat-item {
        display: flex;
        align-items: center;
        gap: 8px;
        background: rgba(0, 0, 0, 0.3);
        padding: 8px 20px;
        border-radius: 40px;
        border: 1px solid var(--pink-core);
        .stat-icon {
          font-size: 1.2rem;
        }
        .stat-label {
          color: var(--blue-glitch);
        }
        .stat-value {
          color: var(--pink-core);
          font-weight: bold;
          font-size: 1.2rem;
        }
      }
      .current-turn {
        border-color: gold;
        .stat-value {
          color: gold;
        }
        &.player-2 .stat-value {
          color: #6ed4ff;
        }
      }
    }
  }

  .canvas-container {
    flex: 1 1 auto;
    width: 100%;
    max-width: 65vh; // 限制最大宽度，保持方形
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .board-canvas {
    display: block;
    width: 100%;
    height: 100%;
    background: transparent;
  }

  .game-controls {
    display: flex;
    gap: 15px;
    justify-content: center;
    margin-top: 20px;
    .control-btn {
      background: rgba(255, 140, 176, 0.1);
      border: 1px solid var(--pink-core);
      border-radius: 30px;
      padding: 8px 20px;
      color: white;
      cursor: pointer;
      transition: all 0.3s;
      font-size: 0.9rem;
      &:hover {
        background: var(--pink-core);
        color: #1a1028;
        box-shadow: 0 0 15px var(--blue-glitch);
      }
    }
  }

  .right-panel {
    width: 0;
    background: rgba(26, 16, 40, 0.9);
    backdrop-filter: blur(10px);
    border: 1px solid var(--pink-core);
    border-radius: 24px 0 0 24px;
    padding: 0;
    display: flex;
    flex-direction: column;
    box-shadow: -10px 0 30px var(--blue-glitch);
    transition: all 0.3s ease;
    overflow-y: auto;
    opacity: 0;
    height: 80vh;
    &.panel-open {
      width: 280px;
      padding: 20px;
      opacity: 1;
      border-radius: 24px;
    }
    .panel-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      h3 {
        margin: 0;
        color: var(--pink-core);
      }
      .close-btn {
        background: transparent;
        border: 1px solid var(--pink-core);
        border-radius: 50%;
        width: 30px;
        height: 30px;
        color: white;
        cursor: pointer;
        &:hover {
          background: var(--pink-core);
          color: #1a1028;
        }
      }
    }
    .panel-content {
      flex: 1;
      overflow-y: auto;
    }
    .ranking-list {
      list-style: none;
      padding: 0;
      margin: 0;
      .ranking-item {
        display: flex;
        align-items: center;
        padding: 10px;
        margin-bottom: 8px;
        background: rgba(255, 255, 255, 0.03);
        border-left: 3px solid transparent;
        border-radius: 8px;
        .rank {
          width: 30px;
          font-weight: bold;
          color: var(--pink-core);
        }
        .name {
          flex: 1;
          font-size: 0.9rem;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
        .score {
          color: var(--blue-glitch);
          font-weight: bold;
        }
        &.rank-1 {
          background: linear-gradient(
            90deg,
            rgba(255, 140, 176, 0.2),
            transparent
          );
        }
        &.rank-2 {
          border-left-color: var(--blue-glitch);
        }
        &.rank-3 {
          border-left-color: #e0a0ff;
        }
      }
    }
    .panel-footer {
      margin-top: 20px;
      text-align: center;
      color: var(--blue-glitch);
      font-size: 0.8rem;
    }
  }

  .game-overlay,
  .rules-overlay {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.7);
    backdrop-filter: blur(10px);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 30;
    .overlay-content,
    .rules-content {
      background: rgba(26, 16, 40, 0.9);
      border: 2px solid var(--pink-core);
      border-radius: 30px;
      padding: 30px;
      max-width: 350px;
      width: 90%;
      text-align: center;
      box-shadow: 0 0 60px var(--blue-glitch);
    }
    .rules-content {
      text-align: left;
      ul {
        padding-left: 20px;
      }
      .highlight {
        color: var(--pink-core);
        font-weight: bold;
      }
      .rule-tip {
        color: var(--blue-glitch);
        margin: 15px 0;
      }
    }
    h2 {
      font-size: 2rem;
      margin: 0 0 20px;
      color: var(--pink-core);
      text-shadow: 0 0 15px var(--pink-core);
    }
    .score-result {
      margin: 20px 0;
    }
    .result-stats {
      display: flex;
      justify-content: center;
      gap: 20px;
      margin-bottom: 15px;
    }
    .score-submit {
      display: flex;
      flex-direction: column;
      gap: 10px;
      input {
        background: rgba(0, 0, 0, 0.5);
        border: 1px solid var(--pink-core);
        border-radius: 30px;
        padding: 10px 15px;
        color: white;
        outline: none;
        &:focus {
          border-color: var(--blue-glitch);
        }
      }
    }
    .game-btn {
      background: linear-gradient(135deg, var(--pink-core), var(--blue-glitch));
      border: none;
      border-radius: 30px;
      padding: 10px 20px;
      color: white;
      font-weight: bold;
      cursor: pointer;
      &:disabled {
        opacity: 0.5;
      }
    }
    .aimis-quote {
      margin-top: 20px;
      color: var(--blue-glitch);
      font-style: italic;
    }
  }

  @media (max-width: 768px) {
    .game-layout {
      padding: 70px 10px 10px;
    }
    .game-header h1 {
      font-size: 1.6rem;
    }
    .game-stats {
      gap: 10px;
    }
    .right-panel.panel-open {
      position: fixed;
      top: 80px;
      right: 0;
      width: 260px;
      border-radius: 0;
      z-index: 20;
    }
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
@keyframes core-pulse {
  0% {
    box-shadow: 0 0 5px var(--pink-core), 0 0 15px var(--blue-glitch);
  }
  50% {
    box-shadow: 0 0 15px var(--pink-core), 0 0 30px var(--blue-glitch);
  }
  100% {
    box-shadow: 0 0 5px var(--pink-core), 0 0 15px var(--blue-glitch);
  }
}
@keyframes grid-flicker {
  0%,
  100% {
    opacity: 0.2;
  }
  50% {
    opacity: 0.3;
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
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
