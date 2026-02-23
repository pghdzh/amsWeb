<template>
  <div class="megumi-page" aria-live="polite">
    <!-- 故障光效背景层 + 数据粒子 + 扫描线 -->
    <div class="glitch-bg"></div>
    <div class="data-particles"></div>
    <div class="scan-line"></div>

    <main class="container" ref="mainContainer">
      <!-- HERO 区域（注入官方剧情信息） -->
      <header class="hero" role="banner">
        <div class="left-col">
          <button class="avatar-card">
            <picture class="avatar-wrap">
              <img
                src="@/assets/images1/slty (11).webp"
                class="avatar"
                loading="lazy"
                decoding="async"
              />
            </picture>
            <div class="avatar-meta">
              <span class="avatar-badge">共鸣能力：长航的星辉</span>
            </div>
          </button>
        </div>
        <div class="right-col" ref="metaCard">
          <div class="hero-panel">
            <h1 class="name glitch-text" data-text="爱弥斯">
              爱弥斯
              <span class="jp">（电子幽灵）</span>
            </h1>
            <p class="subtitle">
              <span class="tag">#星炬学院拉贝尔学部</span>
              <span class="tag">#隧者适格者</span>
              <span class="tag">#飞行雪绒</span>
            </p>
            <p class="lead">
              曾是星炬学院的隧者适格者，如今已成为在星海轻歌的<span
                class="highlight"
                >电子幽灵</span
              >。她在寂静的星海中飞行，星屑在身侧崩解，时间在身后消亡。漫漫孤寂并未消失，它只是被拉伸、稀释、重塑，最终成为她羽翼的一部分。“我知道，只要抬头，那颗星总能找到我。”
            </p>
            <div class="hero-stats">
              <div class="stat-item">
                <span class="stat-label">首次登场</span>
                <span class="stat-value">3.0版本</span>
              </div>
              <div class="stat-item">
                <span class="stat-label">正式实装</span>
                <span class="stat-value">3.1版本</span>
              </div>
              <div class="stat-item">
                <span class="stat-label">共鸣属性</span>
                <span class="stat-value">热熔</span>
              </div>
              <div class="stat-item">
                <span class="stat-label">武器</span>
                <span class="stat-value">迅刀</span>
              </div>
            </div>
          </div>
        </div>
      </header>

      <!-- 新增：官方时间线/彩蛋信息区域 -->
      <section id="timeline-easter" class="card timeline-card">
        <div class="card-head">
          <h2>▌ 官方剧情时间线 & 彩蛋</h2>
          <span class="badge-ems">EMS-DATA</span>
        </div>
        <div class="timeline-grid">
          <div class="timeline-item">
            <div class="timeline-dot"></div>
            <div class="timeline-content">
              <span class="timeline-ver">2.8版本</span>
              <h3>伏笔埋下</h3>
              <p>“那一晚上的失忆以及手的虚化”埋下爱弥斯相关伏笔</p>
            </div>
          </div>
          <div class="timeline-item">
            <div class="timeline-dot"></div>
            <div class="timeline-content">
              <span class="timeline-ver">3.0版本</span>
              <h3>初次登场</h3>
              <p>第一次相遇即是告别</p>
            </div>
          </div>
          <div class="timeline-item">
            <div class="timeline-dot"></div>
            <div class="timeline-content">
              <span class="timeline-ver">3.1版本</span>
              <h3>真相揭晓</h3>
              <p>“我不后悔，但还是...好舍不得你”</p>
            </div>
          </div>
          <div class="timeline-item">
            <div class="timeline-dot"></div>
            <div class="timeline-content">
              <span class="timeline-ver">时间闭环</span>
              <h3>因果循环</h3>
              <p>“...别...难过...”</p>
            </div>
          </div>
        </div>
        <div class="easter-egg">
          <div class="easter-header">
            <span class="easter-icon">🥚</span>
            <span class="easter-title">官方彩蛋 · 摩斯密码</span>
          </div>
          <p class="easter-desc">
            官方OST《以虚无紧系因果》中隐藏摩斯密码，截取后翻译为
            <span class="code">BCAKHOME</span> —— “回家”
          </p>
        </div>
        <div class="easter-egg">
          <div class="easter-header">
            <span class="easter-icon">🎵</span>
            <span class="easter-title">飞行雪绒 · 爱弥斯个人账号</span>
          </div>
          <p class="easter-desc">
            爱弥斯生前以“飞行雪绒”为网名分享原创歌曲，歌友会每年都会筹办
          </p>
        </div>
      </section>

      <!-- 核心机制卡片（保留原有内容） -->
      <section id="personality" class="card personality">
        <div class="card-head">
          <h2>▌ 核心机制</h2>
        </div>
        <ul class="traits">
          <li>
            共鸣模态·震谐下，可额外造成震谐伤害，拥有强力对单伤害能力。队伍中可响应震谐·干涉的角色越多，伤害越高。
          </li>
          <li>
            共鸣模态·聚爆下，可附加【聚爆效应】，拥有强力对群伤害能力。附加【聚爆效应】的频率越高，伤害越高。攻击可获得【同步率】。
          </li>
          <li>
            【同步率】达50%时，在施放普攻第4段后，可消耗【同步率】施放强化合击，获得【共鸣率】。
          </li>
          <li>
            【共鸣率】满时，可施放强化重击充满【同步率】。【同步率】、【共鸣率】都充满时，可施放终结共鸣解放。
          </li>
          <li style="font-weight: bold">输出流程：</li>
          <li>
            基础流程：R-AAA-E【光翼共奏】-F-AA-E【光翼共奏】-Z-R-AAAA（后撤步切人）
          </li>
          <li>
            技能缩写：普攻 = A，重击 = Z，共鸣技能 = E，共鸣解放 = R，声骸 = Q
          </li>
        </ul>
      </section>

      <!-- 共鸣状况（融入电子幽灵设定） -->
      <section id="resonance-status" class="card overview">
        <div class="card-head">
          <h2>▌ 共鸣状况 · 电子幽灵档案</h2>
        </div>
        <div class="two-col">
          <div class="text-block">
            <h3>频谱检验报告 <span class="glitch-tag">▇▂▇数据损毁▇▋▌</span></h3>
            <p>
              「调自深空联合：星炬学院 学生档案」 「共鸣能力检验报告 RA2362-G」
              学生姓名：爱弥斯 是否具有适格者资质：是
              共鸣能力概述：受试样本拉贝尔曲线呈稳定上升态，最终趋向稳定波动，检测结果判断为自然型共鸣者，声痕位于胸口。
              根据入学前提交的个人档案与学生自述，对象▇▇▂▇▋▌▏▉█……<span
                class="glitch-text"
                >很遗憾，这份报告现在已经没有参考价值了，毕竟是生前的记录了~</span
              >就让本人来补充一下吧。现在的我，已经是<span class="highlight"
                >隧者的共鸣者，声痕相比之前也发生了变化，但状态不算很稳定</span
              >。能力……可以显化「隧者兵装」并与之融合，简单来说就是变身啦！当然，为了方便战斗，我也给机兵设计了一套自运转的逻辑，目前模拟配合起来的感受还不错，能够更大限度地利用光炮的覆盖范围。除此之外，我也能以<span
                class="highlight"
                >电子幽灵的形式进入数据系统内部</span
              >。不过，这或许不能称之为共鸣能力的一部分，将之归结于共鸣时的特殊状态带来的……▇▉▇▇▂▇
              “奇怪，这名学生的档案怎么损毁了？打开后都是数据错误。”
              “那个失踪的适格者？嗯……上报给洛瑟菈校长吧。”
            </p>
            <h3>超频诊断报告</h3>
            <p>
              受试样本拉贝尔波形检测图呈椭圆形波动，时域表示稳定，未见任何异常波动倾向。检测结果判断为正常阶段。
              诊断结果：超频临界值正常，稳定性高，暂无超频风险。
              无过往超频史，拉贝尔曲线稳定。 暂无需心理辅导。
              “爱弥斯同学……本学年状态尚处稳定，但我们还是需要更密切地关注她的精神状态。如果情况有变，要及时进行心理干预。”
              “那孩子明明看起来那么开朗……”
              “所以，保持关注就好。既然她希望这样生活，那就相信她的判断，我们作为师长，就做好该做的事吧。”
            </p>
          </div>
        </div>
      </section>

      <!-- 物品卡片（融入剧情意象） -->
      <section id="food" class="card special-section">
        <div class="card-head">
          <h2>▌ 一盘游戏卡带 · 《太空战士卡佳VI》</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/item1.png" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>
              《太空战士卡佳VI》，在索拉里斯的游戏爱好者中为人津津乐道的经典系列最新作。<br />
              沉浸式的角色扮演剧情，丰富有趣的战斗系统，体验成为英雄的孤独之旅，结识十数位可招募的个性伙伴，与宇宙中的邪恶势力进行激烈对抗……除了这些部分，作为彩蛋的那个小游戏也是让爱弥斯极为喜欢的部分，她在通关后经常和家人一起在小屋中联机游玩。<br />
              “哼哼，拼方块的积分排行榜，最后果然还是我排第一！”<br />
              <span class="footnote"
                >※
                官方彩蛋：小屋里面右侧书柜有本《太空战士卡佳VI-星际导航图谱:全星系探索路线与任务图文攻略》</span
              >
            </p>
          </div>
        </div>
      </section>

      <section id="shattered-crystal" class="card special-section">
        <div class="card-head">
          <h2>▌ 隧者手办 · 友情的证明</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/item2.png" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>
              捂在爱弥斯眼上的那双手移开后，面前摆着的就是这样的一个小盒子。<br />
              身边的四位好友笑眯眯地怂恿爱弥斯赶紧拆开，盒中的小型隧者手办就这么露了出来。适格者们像是叽叽喳喳的小动物那样祝贺她生日快乐，希望她能顺利成为那个成功驾驶隧者的人，希望她们能一起去真正的星空……爱弥斯的嘴唇动了动，最终也并未把自己藏在心底深处的那个愿望说出来。<br />
              她珍惜地摩挲了一下这份礼物，站起身来，笑着扑进朋友们的怀里。<br />
              <span class="footnote"
                >※ 那年之后，朋友们再也看不见她，但每年的歌友会依然在举办</span
              >
            </p>
          </div>
        </div>
      </section>

      <section id="cyclic-clock" class="card special-section">
        <div class="card-head">
          <h2>▌ 纸飞机 · 穿越时间的思念</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/item3.png" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>
              爱弥斯坐在学院楼栋顶层，悠然地享受微风吹拂。<br />
              她折过无数纸飞机，但一直记得最初在小屋的那只。午后的光透过窗户照射进来，寒冷的雪和风都被挡在了窗外，只有火炉被烧得劈啪作响的声音。为晚饭炖着的汤散发着甜美醇厚的香气，身边的家人正手指灵巧地为她折出能飞得更远的纸飞机。爱弥斯抬起脸，看到【TA】正微笑着将手中的纸飞机递来。<br />
              那景象在恍惚的回忆中消逝，眼前仍旧是学院的风景。爱弥斯微微一笑，把手中新折的纸飞机投入风中。<br />
              <span class="footnote"
                >※ 纸飞机是贯穿时间线的意象，象征着思念与告别</span
              >
            </p>
          </div>
        </div>
      </section>

      <section id="collective-album" class="card special-section">
        <div class="card-head">
          <h2>▌ 超级豹风雪 · 飞行雪绒的特调</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/food.png" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>
              轻软甜蜜的雪白冰淇淋浮在深色的咖啡上，雪绒豹豹冰雕正安然地泡在其中。在图书馆熬夜苦读时轻轻饮上一口，苦味已被清甜调和，爽口又提神。<br />
              爱弥斯参加星炬学院社团饮品创新大赛的获奖之作，据本人称是表达了作者的思乡之情。<br />
              玩笑话里也有三分真心。在那片冰冷寂静的雪原上，与家人和海豹们玩耍的童年记忆已然相隔遥远，却仍时常停留在她的心头。<br />
              <span class="footnote"
                >※
                小时候，小爱因为把海豹带入家中导致地板湿透，阿漂教训她之后觉得过头买了育儿书籍也在小屋书柜</span
              >
            </p>
          </div>
        </div>
      </section>

      <!-- 共鸣链详解（保留原有内容） -->
      <section
        id="timeline"
        class="card timeline"
        aria-labelledby="timeline-heading"
      >
        <div class="card-head">
          <h2 id="timeline-heading">▌ 共鸣链详解</h2>
        </div>
        <div class="tl-wrap" role="list">
          <div class="vl" aria-hidden="true"></div>
          <ol class="tl-list">
            <li
              v-for="(ev, i) in timeline"
              :key="i"
              class="tl-item"
              role="listitem"
              tabindex="0"
            >
              <div class="tl-card">
                <h3 class="tl-title">第 {{ i + 1 }} 链 · {{ ev.title }}</h3>
                <p class="tl-desc">{{ ev.desc }}</p>
              </div>
            </li>
          </ol>
        </div>
      </section>

      <!-- 爱弥斯纪事（保留原有故事轮播） -->
      <section class="ll-stories" ref="storyRef">
        <div class="card" role="region" aria-labelledby="stories-title">
          <header class="card-head">
            <h2 id="stories-title" class="card-title">▌ 爱弥斯纪事 · 远航星</h2>
          </header>

          <main class="card-body">
            <transition name="fade-slide" mode="out-in">
              <article
                :key="items[current].id"
                :id="`panel-${items[current].id}`"
                class="story"
                role="tabpanel"
                :aria-labelledby="`tab-${items[current].id}`"
              >
                <h3 class="story-head">{{ items[current].title }}</h3>
                <div
                  class="story-content"
                  v-html="items[current].content"
                ></div>
              </article>
            </transition>
          </main>

          <footer class="card-foot">
            <button class="nav-btn" @click="prev" aria-label="上一则故事">
              ‹
            </button>
            <div class="pager">{{ current + 1 }} / {{ items.length }}</div>
            <button class="nav-btn" @click="next" aria-label="下一则故事">
              ›
            </button>
          </footer>
        </div>
      </section>
    </main>

    <!-- 浮动小宠物（电子幽灵分身） -->
    <div class="floating-chibis">
      <img
        v-for="(pet, i) in chibiList"
        :key="i"
        :src="pet.src"
        :style="{ top: pet.top + 'px', left: pet.left + 'px' }"
        class="chibi-img"
      />
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
// 脚本部分基本保持不变，但新增一些数据变量
import { ref, onMounted, nextTick, onBeforeUnmount } from "vue";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

// 爱弥斯语录（融入官方台词）
const messages = [
  "旅途愉快。",
  "别难过。",
  "我会消灭，意图毁灭的恶。",
  "希望你能轻松快乐地活着。",
  "BCAKHOME……回家。",
];
const currentMessage = ref(messages[0]);
const showAiMessage = ref(true);
let msgInterval: number;
function nextMessage() {
  const next = (messages.indexOf(currentMessage.value) + 1) % messages.length;
  currentMessage.value = messages[next];
}

// 原有数据保持不变
const timeline = ref([
  {
    title: "如金粉般洒落的初煦",
    desc: `即刻响应状态下，重击·爱弥斯、重击·机兵暴击伤害提升300%，且蓄力期间可牵引周围的目标。

爱弥斯满足以下条件超过4秒时，获得即刻响应·辉芒状态。
·处于非战斗状态。
·未处于重击·爱弥斯、重击·机兵、共鸣解放星辉破界而来·终结施放状态。
即刻响应·辉芒拥有即刻响应的所有效果，且即刻响应·辉芒不会因星辉破界而来·于此释放状态结束而移除。
处于即刻响应·辉芒状态且不处于星辉破界而来·于此释放状态，施放重击·爱弥斯·二段蓄力或重击·机兵·二段蓄力时，可获得100点【同步率】。

处于共鸣模态·震谐/共鸣模态·聚爆时，爱弥斯自身施放的技能直接造成的伤害击败被附加震谐轨迹/聚爆轨迹状态的敌人时，获得轨迹封存·震谐/轨迹封存·聚爆状态，持续10秒。
轨迹封存·震谐/轨迹封存·聚爆状态下保留击败目标被附加震谐轨迹/聚爆轨迹的最高层数。
爱弥斯下一次自身施放的技能直接造成的伤害立即为命中目标附加对应层数的震谐轨迹/聚爆轨迹，最高可叠加至当前目标的震谐轨迹/聚爆轨迹层数上限，同时清除轨迹封存·震谐/轨迹封存·聚爆状态，1秒内无法再次获得轨迹封存·震谐/轨迹封存·聚爆。`,
  },
  {
    title: "如雪绒般漂浮的音符",
    desc: `共鸣技能光翼共奏·降临的伤害倍率提升100%。
共鸣技能光翼共奏·登台的伤害倍率提升100%。

处于共鸣模态·震谐，共鸣技能光翼共奏额外造成的震谐伤害命中目标时，使目标受到共鸣技能光翼共奏额外造成的震谐伤害倍率提升20%，持续1秒，最多叠加5层。

处于共鸣模态·聚爆获得以下强化：
·星屑共振状态对共鸣技能光翼共奏引爆的【聚爆效应】伤害倍率提升效果增强，对【聚爆效应】主目标的伤害倍率提升效果提升至400%。
·聚爆轨迹对共鸣技能光翼共奏引爆的【聚爆效应】伤害倍率提升效果增强，每层对【聚爆效应】主目标的伤害倍率提升效果提升至15%。
·处于战斗状态，队伍中登场角色附近的敌人被击败时，立即根据【聚爆效应】层数上限引爆【聚爆效应】。`,
  },
  {
    title: "炽烈在静默间延展如初",
    desc: `共鸣解放星辉破界而来·终结的伤害倍率提升100%。
共鸣解放星辉破界而来·过载的伤害倍率提升40%。

处于即刻响应状态，施放重击·爱弥斯、重击·机兵时，根据自身处于共鸣模态·震谐/共鸣模态·聚爆，为附近目标附加【震谐·偏移】/【聚爆效应】。

固有技能星与星之间替换为以下效果：

·处于共鸣模态·震谐时，队伍中的角色附加【震谐·偏移】或造成震谐伤害时，爱弥斯暴击伤害提升60%，共鸣解放星辉破界而来·终结伤害加深25%。
角色编入队伍或切换模态时，重置该效果。

·处于共鸣模态·聚爆时，队伍中的角色附加【聚爆效应】时，爱弥斯暴击伤害提升60%，共鸣解放星辉破界而来·终结伤害加深25%。
角色编入队伍或切换模态时，重置该效果。`,
  },
  {
    title: "于无垠电子海间轻舞",
    desc: `施放变奏技能以旋律穿越长空、变奏技能携星辉降临于此、共鸣技能合击·突刺、共鸣技能光翼共奏时，队伍中的角色全属性伤害加成提升20%，持续30秒。`,
  },
  {
    title: "远航至那星海尽处",
    desc: `爱弥斯自身技能直接造成的伤害击败目标时，【流溢辉光】重置为100%。

爱弥斯受到致命伤害时，将失去意识并进入二维电子幽灵状态，持续5秒。
进入二维电子幽灵状态时，为队伍中的角色提供爱弥斯360%攻击的护盾，持续5秒。退出二维电子幽灵状态时，爱弥斯将恢复意识并回复100%生命值与30点共鸣能量。该效果每10分钟可触发1次。
爱弥斯恢复意识时，退出二维电子幽灵状态并移除该效果提供的护盾。`,
  },
  {
    title: "春风祝颂你的旅途",
    desc: `目标受到爱弥斯的共鸣解放伤害提升40%。

处于共鸣模态·震谐时，爱弥斯的震谐伤害可暴击，暴击固定为80%，暴击伤害固定为275%。

处于共鸣模态·聚爆，并处于战斗状态，队伍中登场角色附近的敌人受到聚爆效应触发的伤害可暴击，暴击固定为80%，暴击伤害固定为275%。

共鸣回路为寂静赋形为目标附加震谐轨迹、聚爆轨迹层数翻倍。

处于共鸣模态·震谐/共鸣模态·聚爆，并处于战斗状态，队伍中登场角色附近的敌人震谐轨迹/聚爆轨迹层数上限提升至60层。爱弥斯施放共鸣技能光翼共奏期间，对范围内目标附加10层震谐轨迹/聚爆轨迹，持续30秒。`,
  },
]);

const reducedMotion =
  window.matchMedia &&
  window.matchMedia("(prefers-reduced-motion: reduce)").matches;

const items = [
  {
    id: "1",
    title: "在雪原上",
    content: `雪落在爱弥斯的鼻尖上，湿润的凉意让她感觉有些痒痒的，她盯着湖面看了一会儿，最终还是决定放弃玩水的打算。

前段时间，她耗费心思努力从湖里面捞出来了几根硕大的狐尾蒲藻当做武器，和冰原上的雪绒海豹们玩耍。在这场大战中她扮演英雄，海豹们扮演坏蛋——考虑到这群慵懒温和的生物爬行速度很慢，基本都是爱弥斯大获全胜。可惜那次玩过了头，狐尾蒲藻的碎块使小屋狼藉满地，那个人回家之后，在门口沉默了很久。

自然，爱弥斯挨了一顿训。

事后那个人似乎对此有些愧疚，觉得自己对孩子凶过了头，又买了一堆卡带作为补偿和应对措施。这招相当有效，电子游戏的魅力迅速俘虏了爱弥斯，她就暂时把在现实里大战的事儿抛之脑后了。而在所有的卡带里，她最喜欢的是《太空战士卡佳》系列。在宇宙中一边旅行一边拯救世界？太棒了！谁不想扮演一个忧郁、帅气又意志坚定的英雄呢？

爱弥斯推开门，那个人正坐在沙发上看书，她高高兴兴地挤过去，举起手柄。和我玩！女孩的动作传达着这一讯号，于是对方笑着刮了刮女孩的鼻尖，欣然应允。
冬日漫长，但此刻炉火燃烧，一切都温暖平静。

游戏进行了一段时间，爱弥斯说，今天的任务就做到这里吧，我和你说，我在地图右侧找到了一个隐藏房间呢！密码是9072，开门之后可以玩拉海洛方块，可好玩儿了。那个人问，爱弥斯，你不是昨天还着急继续推任务拯救世界吗？对，她摇晃着脑袋笑着说，但今天我又不想了……而且，要是任务做完了，游戏也就结束了……我还想和你一起玩更久呢。就算是拯救世界，也是需要休息的，对吧？你要学会劳逸结合啦！

那只温暖的手揉了揉她的发顶，带着一丝奇妙的感慨低声道：你说得对，爱弥斯。我会试着这么做的。

那……要从现在做起噢？

好，从现在做起。那个人笑了笑。其实……现在我在这里，已经算是在休息。来吧，我们玩拉海洛方块，你先。`,
  },
  {
    id: "2",
    title: "年轻人们",
    content: `虚质逆向工程学的成绩出分的下午，爱弥斯和她的朋友们例常的午饭时间一片愁云惨淡。

埃拉拉一头鲜艳的红发都有些黯淡，虚弱地哀鸣着，啊，连食堂的饭都不香了……！在她身边认真咀嚼一碗不明物质的诺娃则说，朋友，虽然这分数的确让人难受，但我们不能本末倒置——这食堂的饭它本来也不香啊。琳正因为分数大受打击，塞莱斯特在温声安慰她，爱弥斯则以神游天外的表情吸了一口她特调过口味的嘻嘻果冻，和身边几位关系融洽的适格者们进行着漫谈。

埃拉拉突然扭头看她：爱弥斯！你、你一点都不着急吗？

她眨巴眼睛：着急什么？……啊，分数？但这次大家成绩不都差不多嘛，萨迦教授判分太严格啦，又不是大家真的不行，没事没事。说到一半，爱弥斯拿出纸巾递给她们中性格最柔软的那位。啊，小琳，擦擦泪——！

塞莱斯特接了过去，和其他几个人对视一眼，轻声道：你好像不太在意作为适格者的未来。

她平常会怎么说呢？大约会找几个话题打打闹闹地糊弄过去，但今天是那个人的生日，爱弥斯正在思念对方，因此有些忘记在朋友们前掩饰情绪。适格者……也只是一种可能性嘛，她语调平淡地说，比起这个，有人曾说更希望我能轻松快乐地生活，我也觉得这样比较好。我的理想没有大家那么厉害啦，懒散的人松弛点也很正常吧？

真的吗？朋友们望着她的眼睛问。

爱弥斯只是回以微笑：真的。

但并非如此，只是因为那个理想太过沉重、太过傲慢、太过荒谬，因此不能在平稳的词句中被宣之于口。它每一日都比前一日成长得更加膨大，在爱弥斯的心灵中扎入根须，但她视之不见，因为她已经接受了那句嘱托。如果日复一日的快乐就是好的，那就让她这么一直快乐下去吧！她会信守承诺，除非……
思绪被打断，埃拉拉在她的脸颊上狠狠捏了一把。

怎么啦？爱弥斯皱着脸含混不清地说道。

心口不一的坏人，她的朋友抱怨道，好了好了不问你了，下午图书馆复习一起去吗？

她点头道：当然~

爱弥斯笑着和朋友们打闹作一团，慢悠悠走向电梯。她们都还很年轻，并不急于一时去寻找答案，在这所满载着天才的学院中，拥有无限可能的未来。`,
  },
  {
    id: "3",
    title: "唯不可见者可见",
    content: `爱弥斯走到镜子前，镜面并未映照出她的影子。

在她死后，整个世界就与之错开了。适格者们曾讨论过人的本质是什么，埃拉拉说是爱，诺娃说是记忆，塞莱斯特说是自我，琳说是信仰……不对，爱弥斯想，人的本质是频率。以她现在的情况而言，她只能这么去理解。是因为和隧者共鸣，因此她的存在性质也被改变了？她的躯体已经在模拟驾驶舱中被撕碎，她应该死了，却仍旧存在……不过，她现在已经不再去想存在的本质是什么这种问题了。尽管对一个电子幽灵而言，漫长的时间足够去思考任何问题。

但爱弥斯现在知道了更重要的事。

在她进入高悬天际的隧者炉芯后，她看到了藏在其中的那条讯息。

原来是这样，她想，原来是这样。所以这条讯息从未被读到，因为在最初它就已经被放置在了无法被查阅的地方，只有此时身为隧者共鸣者的她才能看见。整个地下空洞都仰赖着这“太阳”生存，拉海洛人自然不会有机会知晓。她该高兴吗？以这条讯息的内容而言，她将有机会实现自己的愿景——但这的确又像个阴差阳错的玩笑，因为那并不是她想象中的实现方式。可这是只有她能做到的事。

如果有我能做的事，我就会去做。

……对，是该这样。爱弥斯想起那个人，在心里为自己加油。她觉得自己可以再勇敢些，再坚定些。当然，现在还不够，但没关系，还有时间去做准备。从这方面而言，她已经受到了命运的优待。想着想着，她不禁轻轻哼起歌来。可惜，飞行雪绒的曲子没法再更新了，如果能预料到明天，她会写完那首歌的。

镜面没有映出她的面孔，因此爱弥斯并未看到，她脸上的微笑像是被一划被蜡笔涂歪的痕迹，在末端戛然而止。`,
  },
  {
    id: "4",
    title: "问我何所惧，问我何所忧",
    content: `寂静。

她转过身，看到了那只“眼睛”。啊，是的，形似眼睛，但爱弥斯知道那不是。黑洞？也许人类可以用自己所知的东西去这么称呼，但在她的眼中，在已经与隧者共鸣的她眼中，那是某种在不断漫溢发生的“现象”。被其吞噬和笼罩的东西失去存在，深空联合称其为虚质——那就这么叫吧，对于处在比之更低维度的生物而言，总要有个称呼。这就和人类叫它“阿列夫一”是同样的道理。

爱弥斯已经不止一次和它“对视”了。对于在虚质空间和外部不断往返的她而言，这是必然会遭遇的事，只能习惯。

直到前不久为止，她都还是会受到一定的影响。虚无，对死亡的渴望，无意义，寂静的宇宙……凝视阿列夫一带来的负面影响，就连身为死者的她都难以避免。但现在，在爱弥斯于那被吞噬的文明的残骸中探索后，她看着那只巨大的，超越了时间与空间的眼睛，只听到了自己趋近疯狂的大笑声。
这就是那个人一直追寻的东西吗？笑声从她的喉咙中沉闷地响起。这就是真相？

真荒谬。

一切自年幼时萌发的愿景和理想，意义都因此消解。爱弥斯忽然觉得很累，但在疲惫之后，又有一丝萦绕不去的伤心和担忧钻进她的心魂。那个人该怎么办呢？那个人知道这些事吗？如果知道，爱弥斯不敢想象对方一直以来是抱持着何种感情在世间行走。如果不知道……那她就应该把这一切都藏起来，都砸碎，都销毁……她的心中一半是冰冷的担忧，一半是燃烧的怒火。生平第一次，她知晓了何为仇恨。

那只被爱弥斯造出的纸飞机幻影，在她的掌心被揉皱成一团纸屑，落向无尽的虚空。`,
  },
  {
    id: "5",
    title: "一切未曾说出的",
    content: `爱弥斯慢慢踏着楼梯向上，回到了桌边。

因为刚才那个梦醒来后，她也睡不着了。事实上，她本就无需睡眠，梦也更像是记忆以另一种形式进行重播。明天她和【TA】就会抵达隧者脚下，她不太确定会发生什么，因此想要做好准备。

话语是意义的准备，因此她想把一些想和【TA】说，却在此刻还无法宣之于口的话记录来下。

该说些什么好呢？也许应该把这么多年发生的事情，都告诉对方。大事小事，欢乐和忧伤，无关紧要的一切，关系世界命运的一切……或者，也可以小小的责备对方一下，因为时间已经过去了很久，对方缺席了太久，她要分享的东西也堆积得太多。如果这些都不太对，那么就把这次公路片之旅画下来，就像以前那样。她说了又删，删了又说，最后只留下了一小段内容。

“我知道，只要抬头，那颗星总能找到我。”

爱弥斯在房间中的电子设备里留下了这段话，等待它在更久一些之后的时间被发现，或者永远不被发现。

此刻，她也的确没有更多想要的东西了。

【TA】现在看起来还挺快乐的，这比任何事都要好。如果可以，那就永远隐瞒下去吧，只要那个人能得到幸福，其他事情她都可以付之一笑。她可以做个自私的人，也可以做个无私的人，可面对家人，人难免都会护短，爱弥斯也不能免俗。真相又有什么重要的呢？

只要【TA】能自由而快乐地活着就好。

爱弥斯想着想着，忽然意识到这和过去束缚过她的那句嘱托并无区别。她忍不住笑了一下。此刻万籁俱寂，雪无声地落在天地间，于消融前等待着春日的降临。`,
  },
];
const current = ref(0);

function select(index) {
  current.value = index;
  const id = `tab-${items[index].id}`;
  const el = document.getElementById(id);
  if (el) el.focus();
}
const storyRef = ref<HTMLElement | null>(null);

function prev() {
  current.value = (current.value - 1 + items.length) % items.length;
  select(current.value);
  storyRef.value.scrollIntoView({ behavior: "smooth", block: "start" });
}

function next() {
  current.value = (current.value + 1) % items.length;
  select(current.value);
  storyRef.value.scrollIntoView({ behavior: "smooth", block: "start" });
}

interface Chibi {
  src: string;
  top: number;
  left: number;
}

const chibiList = ref<Chibi[]>([]);

onMounted(async () => {
  // 初始化语录轮换
  msgInterval = window.setInterval(() => {
    nextMessage();
  }, 8000);

  if (!reducedMotion) {
    const tl = gsap.timeline({ defaults: { ease: "power3.out" } });
    tl.from(
      ".right-col > .hero-panel .name",
      { duration: 0.9, x: 24, opacity: 0 },
      "-=0.45"
    );

    gsap.utils.toArray(".tl-item").forEach((it: any, idx: number) => {
      gsap.from(it.querySelector(".tl-card"), {
        scrollTrigger: {
          trigger: it,
          start: "top 85%",
          toggleActions: "play none none reverse",
        },
        opacity: 0,
        y: 22,
        duration: 0.7,
        delay: idx * 0.06,
      });
    });
  } else {
    document.documentElement.classList.add("reduced-motion");
  }

  // 浮动小宠物（与之前相同）
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
.megumi-page {
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
  position: relative;
  min-height: 100vh;
  background: var(--bg-deep);
  font-family: "JetBrains Mono", "Fira Code", "PingFang SC", monospace;
  color: var(--white-glow);
  -webkit-font-smoothing: antialiased;
  padding: clamp(20px, 4vw, 80px);
  box-sizing: border-box;
  padding-top: 60px;

  &.reduced-motion * {
    transition: none !important;
    animation: none !important;
  }

  .container {
    position: relative;
    z-index: 10;
    max-width: 1100px;
    margin: 0 auto;
    padding: clamp(12px, 2vw, 40px);
  }
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

/* 浮动小宠物 */
.floating-chibis {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 99;
  .chibi-img {
    position: absolute;
    width: 80px;
    user-select: none;
    transform-origin: center center;
    pointer-events: auto;
    z-index: 10;
    filter: drop-shadow(0 0 10px var(--pink-glow))
      drop-shadow(0 0 20px var(--blue-glitch));
    opacity: 0.8;
    transition: filter 0.3s;
    &:hover {
      filter: drop-shadow(0 0 20px var(--pink-core))
        drop-shadow(0 0 40px var(--blue-glow));
    }
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

/* HERO区域 */
.hero {
  position: relative;
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 28px;
  align-items: start;
  padding: 12px;
  margin-bottom: 24px;
  z-index: 5;

  &::before {
    content: "";
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;
    background-image: repeating-linear-gradient(
      to bottom,
      rgba(255, 255, 255, 0.02) 0px,
      rgba(255, 255, 255, 0.02) 1px,
      transparent 1px,
      transparent 16px
    );
    opacity: 0.06;
    mix-blend-mode: overlay;
    transform: translateY(-4%);
    animation: staff-drift 16s linear infinite;
  }

  .left-col {
    display: flex;
    flex-direction: column;
    align-items: center;
    z-index: 6;
  }

  .avatar-card {
    background: rgba(26, 16, 40, 0.6);
    backdrop-filter: blur(10px);
    border-radius: 16px;
    padding: 12px;
    display: grid;
    justify-items: center;
    gap: 8px;
    border: 2px solid transparent;
    background-clip: padding-box;
    box-shadow: 0 0 30px var(--blue-glitch),
      inset 0 0 20px rgba(255, 140, 176, 0.1);
    animation: cursorAnimation_link 1s infinite step-start;
    transition: all 0.3s;
    width: 100%;
    position: relative;
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

  .avatar-wrap {
    width: 100%;
    display: block;
  }

  .avatar {
    width: 100%;
    height: auto;
    border-radius: 12px;
    display: block;
    box-shadow: 0 0 30px var(--blue-glitch);
    filter: saturate(0.9) contrast(0.95) brightness(0.9);
    transition: all 0.3s;
  }

  .avatar-card:hover .avatar {
    transform: translateY(-4px) scale(1.02);
    filter: saturate(1) contrast(1) brightness(1);
  }

  .avatar-meta {
    display: flex;
    gap: 8px;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
  }

  .avatar-badge {
    font-size: 0.9rem;
    color: var(--white-glow);
    background: rgba(255, 140, 176, 0.1);
    padding: 6px 10px;
    border-radius: 999px;
    border: 1px solid var(--pink-core);
    font-weight: 600;
  }

  .right-col {
    display: flex;
    flex-direction: column;
    gap: 12px;
    z-index: 6;
  }

  .hero-panel {
    position: relative;
    z-index: 6;
    padding: 24px;
    border-radius: 16px;
    background: rgba(26, 16, 40, 0.6);
    backdrop-filter: blur(10px);
    border: 1px solid var(--pink-core);
    box-shadow: 0 0 30px var(--blue-glitch),
      inset 0 0 20px rgba(255, 140, 176, 0.1);
  }

  .name {
    font-size: clamp(1.8rem, 3vw, 2.8rem);
    margin: 0;
    font-weight: 900;
    letter-spacing: 1px;
    line-height: 1;
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

  .jp {
    font-size: 0.95rem;
    color: var(--blue-glow);
    margin-left: 8px;
    font-weight: 400;
  }

  .subtitle {
    margin: 6px 0 0;
    font-size: 1rem;
    color: var(--white-glow);
    font-weight: 600;
    line-height: 1.4;
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
    .tag {
      background: rgba(110, 212, 255, 0.1);
      border: 1px solid var(--blue-glitch);
      padding: 2px 8px;
      border-radius: 20px;
      font-size: 0.85rem;
    }
  }

  .lead {
    margin: 10px 0 0;
    color: var(--white-glow);
    opacity: 0.96;
    line-height: 1.7;
    max-width: 72ch;
    .highlight {
      color: var(--pink-core);
      text-shadow: 0 0 8px var(--pink-core);
    }
  }

  .hero-stats {
    display: flex;
    gap: 16px;
    margin-top: 16px;
    flex-wrap: wrap;
    .stat-item {
      background: rgba(255, 140, 176, 0.1);
      border: 1px solid var(--pink-core);
      border-radius: 12px;
      padding: 8px 12px;
      display: flex;
      flex-direction: column;
      align-items: center;
      .stat-label {
        font-size: 0.75rem;
        color: var(--blue-glow);
        text-transform: uppercase;
      }
      .stat-value {
        font-size: 1rem;
        font-weight: bold;
        color: var(--pink-core);
      }
    }
  }
}

/* 通用卡片 */
.card {
  margin-top: 24px;
  background: rgba(26, 16, 40, 0.6);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 0 30px var(--blue-glitch),
    inset 0 0 20px rgba(255, 140, 176, 0.1);
  color: var(--white-glow);
  border: 1px solid var(--pink-core);
  transition: all 0.3s;
  position: relative;
  overflow: hidden;

  &:hover {
    transform: translateY(-4px);
    box-shadow: 0 0 40px var(--pink-core), 0 0 60px var(--blue-glitch);
  }

  .card-head {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    margin-bottom: 16px;
    h2 {
      margin: 0;
      font-size: 1.3rem;
      color: var(--pink-core);
      font-weight: 800;
    }
    .badge-ems {
      background: rgba(110, 212, 255, 0.2);
      border: 1px solid var(--blue-glitch);
      padding: 4px 8px;
      border-radius: 20px;
      font-size: 0.75rem;
      color: var(--blue-glow);
    }
  }
}

/* 时间线与彩蛋卡片 */
.timeline-card {
  .timeline-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 16px;
    margin-bottom: 24px;
    @media (max-width: 720px) {
      grid-template-columns: 1fr;
    }
  }
  .timeline-item {
    display: flex;
    gap: 12px;
    align-items: flex-start;
    .timeline-dot {
      width: 12px;
      height: 12px;
      background: var(--pink-core);
      border-radius: 50%;
      margin-top: 4px;
      box-shadow: 0 0 15px var(--pink-core);
    }
    .timeline-content {
      flex: 1;
      .timeline-ver {
        font-size: 0.75rem;
        color: var(--blue-glow);
        background: rgba(110, 212, 255, 0.1);
        padding: 2px 6px;
        border-radius: 4px;
      }
      h3 {
        margin: 4px 0;
        font-size: 1rem;
        color: var(--white-glow);
      }
      p {
        margin: 0;
        font-size: 0.9rem;
        color: rgba(255, 255, 255, 0.8);
      }
    }
  }
  .easter-egg {
    background: rgba(255, 140, 176, 0.05);
    border: 1px dashed var(--pink-core);
    border-radius: 12px;
    padding: 12px;
    margin-bottom: 12px;
    .easter-header {
      display: flex;
      align-items: center;
      gap: 8px;
      margin-bottom: 8px;
      .easter-icon {
        font-size: 1.2rem;
      }
      .easter-title {
        font-weight: bold;
        color: var(--pink-core);
      }
    }
    .easter-desc {
      margin: 0;
      font-size: 0.9rem;
      .code {
        background: rgba(0, 0, 0, 0.3);
        padding: 2px 6px;
        border-radius: 4px;
        color: var(--blue-glow);
        font-family: monospace;
      }
    }
  }
}

/* personality 列表 */
.personality .traits {
  list-style: none;
  padding: 0;
  margin: 0;
  li {
    padding: 12px 0;
    border-bottom: 1px dashed rgba(255, 140, 176, 0.2);
    font-size: 1rem;
    color: var(--white-glow);
  }
}

/* overview 双栏 */
.overview .two-col {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
  margin-top: 12px;
}
.overview .text-block {
  flex: 1 1 100%;
  line-height: 1.6;
  color: var(--white-glow);
  h3 {
    color: var(--pink-core);
    margin: 16px 0 8px;
    display: flex;
    align-items: center;
    gap: 8px;
    .glitch-tag {
      font-size: 0.75rem;
      background: rgba(255, 0, 0, 0.2);
      color: #ff6b85;
      padding: 2px 6px;
      border-radius: 4px;
    }
  }
  .glitch-text {
    color: #ff6b85;
    animation: glitch 2s infinite;
  }
  .highlight {
    color: var(--pink-core);
    text-shadow: 0 0 8px var(--pink-core);
  }
}

/* special-section 卡片 */
.special-section {
  .content {
    display: flex;
    gap: 24px;
    align-items: center;
    margin-top: 16px;
    @media (max-width: 720px) {
      flex-direction: column;
    }
  }
  .img-area {
    flex: 0 0 200px;
    img {
      width: 100%;
      height: auto;
      border-radius: 12px;
      box-shadow: 0 0 30px var(--blue-glitch);
      border: 1px solid var(--pink-core);
    }
  }
  .text-block {
    flex: 1;
    line-height: 1.6;
    color: var(--white-glow);
    .footnote {
      display: block;
      margin-top: 12px;
      color: var(--blue-glow);
      font-size: 0.85rem;
      border-left: 3px solid var(--pink-core);
      padding-left: 12px;
    }
  }
}

/* timeline 共鸣链 */
.timeline {
  position: relative;
  .tl-wrap {
    position: relative;
    padding: 24px 12px 12px 96px;
  }
  .vl {
    position: absolute;
    left: 80px;
    top: 32px;
    bottom: 20px;
    width: 4px;
    border-radius: 6px;
    background: linear-gradient(180deg, var(--blue-glitch), var(--pink-core));
    box-shadow: 0 0 20px var(--blue-glitch);
    opacity: 0.95;
  }
  .tl-list {
    list-style: none;
    margin: 0;
    padding: 0;
  }
  .tl-item {
    position: relative;
    display: flex;
    gap: 18px;
    align-items: flex-start;
    margin: 20px 0;
    transition: all 0.3s;
  }
  .tl-card {
    background: rgba(26, 16, 40, 0.6);
    backdrop-filter: blur(4px);
    border-radius: 12px;
    padding: 16px 20px;
    box-shadow: 0 0 20px var(--blue-glitch);
    width: calc(100% - 130px);
    border: 1px solid var(--pink-core);
    transition: all 0.3s;
    position: relative;
    z-index: 2;
    &:hover {
      transform: translateY(-4px);
      box-shadow: 0 0 30px var(--pink-core);
    }
  }
  .tl-title {
    margin: 0 0 6px 0;
    font-size: 1.1rem;
    color: var(--pink-core);
  }
  .tl-desc {
    margin: 0;
    color: var(--white-glow);
    white-space: pre-wrap;
    font-size: 0.9rem;
  }
  @media (max-width: 980px) {
    .tl-wrap {
      padding-left: 14px;
    }
    .vl {
      display: none;
    }
    .tl-card {
      width: 100%;
    }
  }
}

/* 爱弥斯纪事 */
.ll-stories {
  .card {
    border-radius: 16px;
    color: var(--white-glow);
    overflow: hidden;
  }
  .card-head {
    padding: 16px 20px;
    border-bottom: 1px solid rgba(255, 140, 176, 0.2);
    .card-title {
      font-size: 1.3rem;
      color: var(--pink-core);
    }
  }
  .card-body {
    padding: 20px;
    min-height: 200px;
  }
  .story {
    .story-head {
      font-size: 1.2rem;
      color: var(--blue-glow);
      margin-bottom: 12px;
    }
    .story-content {
      line-height: 1.7;
      color: var(--white-glow);
      white-space: pre-wrap;
    }
  }
  .card-foot {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 12px 20px;
    border-top: 1px solid rgba(255, 140, 176, 0.2);
    .nav-btn {
      background: transparent;
      border: 1px solid var(--pink-core);
      color: var(--white-glow);
      width: 36px;
      height: 36px;
      border-radius: 50%;
      animation: cursorAnimation_link 1s infinite step-start;
      transition: all 0.3s;
      &:hover {
        background: var(--pink-core);
        color: #1a1028;
        box-shadow: 0 0 15px var(--blue-glitch);
      }
    }
    .pager {
      color: var(--blue-glow);
    }
  }
  .fade-slide-enter-active,
  .fade-slide-leave-active {
    transition: all 0.3s ease;
  }
  .fade-slide-enter-from {
    opacity: 0;
    transform: translateY(10px);
  }
  .fade-slide-leave-to {
    opacity: 0;
    transform: translateY(-10px);
  }
}

/* 动画 */
@keyframes staff-drift {
  0% {
    transform: translateY(-4%);
  }
  50% {
    transform: translateY(4%);
  }
  100% {
    transform: translateY(-4%);
  }
}

/* 响应式 */
@media (max-width: 980px) {
  .hero {
    grid-template-columns: 1fr;
  }

  .avatar-card {
    width: 200px;
  }
}

@media (max-width: 720px) {
  .megumi-page {
    padding: 12px;
    padding-top: 60px;
  }
  .hero-stats {
    justify-content: center;
  }
  .ai-message {
    bottom: 10px;
    right: 10px;
    font-size: 11px;
    padding: 4px 10px;
  }
}
</style>
