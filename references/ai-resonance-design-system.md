# AI-Resonance 设计系统工程化剥离文档(附同类方案全景)

> 生成于 2026-10-12。素材:WZZNNE/AI-Resonance(main 分支,一手抓取 tokens.css 21KB / presets.css / VISUAL.md 21KB 全文 /
> engine.ts / boot.js / UI.md),生态数据(GitHub API + Web 检索,星数与状态均当日核实)。
> 第一部分把它的设计系统当作一个软件工件完整解剖,目标是**可复建**;第二部分回答"有没有相同方案"。
> 姊妹件:`ai-resonance-VISUAL.md`(规范原文)、portal-v3 `docs/VISUAL.md`(我们按同范式落地的契约)。

---

# 第一部分 设计系统作为软件工程:五层解剖

AI-Resonance 的设计系统不是一个 style 文件夹,而是**一个有契约、有分层、有引擎、有逃生舱、有测试的小型软件系统**。
全部落在一个 pnpm 包 `packages/web` 里,运行时零依赖(纯 CSS + 少量 TS)。

## 1. 五层架构总览

```
L4 用户层    public/custom.css(fork 覆盖) · Settings › Appearance · window.ResonanceTheme · 主题文件导入/导出 JSON
L3 引擎层    theme/engine.ts(accent 色彩数学/安全模式/token 覆盖) · theme/prefs.ts(应用偏好) · public/boot.js(防闪白)
L2 规则层    ui/ui.css(材质规则) · ui/hud.css(HUD) · ui/motion.css(动效) · ui/glass.ts(指针光斑)
L1 Token 层  theme/tokens.css(角色色 + 派生材质,21KB) · theme/presets.css(预设:只换值)
L0 契约层    docs/VISUAL.md(视觉语言规范,"这里的值就是代码里的值")
```

依赖方向自上而下单向:改视觉**先改 L0**,再落到 L1/L2;L3 引擎保证 L4 用户操作永远不破坏 L1–L2;
`?safe=1` 逃生舱可以在任何时刻关掉 L4 的全部自定义,回到内置状态修砖。

**为什么不会腐化**——三层防线:
1. L0 契约:"If code and this file disagree, fix one of them in the same change."(文档与代码不一致,同一次提交修掉一个)
2. L1 派生:材质全部从色彩 token **计算**得出,预设没有机会各写各的阴影/边框;
3. L3 逃生舱:任何用户的主题灾难都能一键回退。

## 2. Token 分层模型(L1 内部的四层)

### L1.0 色彩角色(Role)
`--bg / --bg-2`(页面)· `--surface / --surface-2 / --surface-3`(表面三级)· `--well`(凹槽)·
`--line / --line-2`(发丝线)· `--text / --text-2 / --text-3`(文字三档)· `--accent / --accent-fill /
--accent-2 / --accent-ink`(强调色四件套)· `--signal / --signal-2 / --signal-line`(全站唯一信号色)·
`--ok / --warn / --danger / --info / --live`(状态)· `--scrim`(遮罩)· `--knob`(旋钮白)

### L1.1 语义色(Meaning)
色彩即含义,不做装饰:**五榜单色** `--hue-repos/papers/news/social/labs`、**八分类色** `--cat-release/product/
research/tool/engineering/discussion/industry/policy`、**八信号色** `--sig-1..8`(评分条分段)。
卡片把自己的榜单色设为局部 `--hue`,组件从环境读取——换榜单不用改组件。

### L1.2 材质(Material)——派生,不是手写
`--panel-*`(内容玻璃)· `--plate-*`(小件)· `--well-*`(凹槽)· `--glass-*`(铬件)· `--press`(按下)·
`--thumb-*`(凸起滑块)· `--elev-0..3`(投影四档)· `--tint-1..3`(玻璃内嵌填充)· `--rim`(折射边)·
`--hl`(彩色填充顶光)· `--spill`(页面双光源)· `--grid / --scan / --grain / --vignette`(场景层)。
**全部用 `color-mix()` 从 L1.0 计算**,这是整个系统最核心的一步(配方见 §3)。

### L1.3 状态与预算(Budget)
`--glow / --ring / --ring-inset / --bloom`(辉光预算)· `--hud / --hud-hot / --hud-faint / --hud-glow / --ticks`
(HUD 层)· `--ease / --ease-pop / --dur-1/2/3`(动效)· `--font-scale`(0.85–1.3)· `--density`。
token 目录在 engine.ts 头注释里维护着完整清单(11 大类、100+ 变量,含 legacy 别名),本身就是文档。

## 3. 材质派生配方(可直接搬走的 CSS)

光照模型一句话:**固定左上光源,每个材质都有"顶光唇 + 边缘 + 脚部阴影"**。"没有光照的半透明不允许存在"(禁止模糊矩形)。

```css
/* Panel — 内容玻璃:磨砂 + 折射 + 三段投影 */
--panel-tint: color-mix(in srgb, var(--surface) 52%, transparent);
--panel-bg:
  radial-gradient(120% 70% at 0% 0%, rgb(255 255 255 / 0.06), transparent 55%),  /* 左上顶光 */
  linear-gradient(180deg,
    rgb(255 255 255 / 0.075) 0, rgb(255 255 255 / 0.018) 72px,                   /* 顶部内光衰减 */
    transparent 40%,
    rgb(0 0 0 / 0.18) 100%),                                                      /* 脚部变暗 */
  var(--panel-tint);
--panel-edge:                       /* 全部用 inset 阴影拼边,不用 border → 不占布局 */
  inset 0 1px 0 rgb(255 255 255 / 0.17),   /* 高光唇 */
  inset 0 -1px 0 rgb(255 255 255 / 0.05),  /* 底缘反光 */
  inset 0 0 0 1px rgb(255 255 255 / 0.05), /* 1px 玻璃边 */
  inset 0 0 20px rgb(255 255 255 / 0.035), /* 边内透光(厚度感) */
  inset 0 -16px 26px -20px rgb(0 0 0 / 0.55);
--panel-drop: 0 1px 1px rgb(0 0 0/.45), 0 14px 28px -14px rgb(0 0 0/.6), 0 42px 84px -38px rgb(0 0 0/.85);
--panel-blur: blur(26px) saturate(1.6);

/* Plate — 小实体件(按钮/chip/键帽):近实心 + 顶光 */
--plate-bg:
  linear-gradient(180deg, rgb(255 255 255/.09) 0%, rgb(255 255 255/.025) 46%, rgb(0 0 0/.08) 100%),
  color-mix(in srgb, var(--surface-2) 86%, transparent);
--plate-edge: inset 0 1px 0 rgb(255 255 255/.13), inset 0 -1px 0 rgb(0 0 0/.35), inset 0 0 0 1px rgb(255 255 255/.06);

/* Well — 凹槽(输入框/分段轨道/进度槽):切进表面的沟 */
--well-bg: color-mix(in srgb, var(--well) 62%, transparent);
--well-edge: inset 0 1px 2px rgb(0 0 0/.6), inset 0 0 0 1px rgb(0 0 0/.4), 0 1px 0 rgb(255 255 255/.06);

/* Glass — 铬件(顶栏/标签栏/弹层):比 Panel 更稠 */
--glass-bg: linear-gradient(180deg, rgb(255 255 255/.05), transparent 60%), color-mix(in srgb, var(--surface) 68%, transparent);
--glass-blur: blur(28px) saturate(1.8);

/* Press — 按下态:内凹 + 缩放 */
--press: inset 0 2px 5px rgb(0 0 0/.55), inset 0 0 0 1px rgb(0 0 0/.4);  /* + transform: scale(0.98) */

/* Spill — 页面场景(玻璃背后的可折射物):双弱光源,刻意压暗 */
--spill:
  radial-gradient(70vmax 55vmax at 6% 108%, color-mix(in srgb, var(--signal) 8%, transparent), transparent 62%),
  radial-gradient(62vmax 50vmax at 98% -10%, color-mix(in srgb, var(--hue-social) 9%, transparent), transparent 60%);
```

三条硬规矩:
1. **预设只许换 L1.0/L1.1 的颜色 + 字体 + 圆角,材质一个字不许写**——换一套色 = 自动得到一整套深度系统;
2. plate/panel **不用 `border` 属性**(边全部在 box-shadow 里),hover/按下永不跳布局;
3. 玻璃内部 `--surface-2/3` 自动解析为 `--tint-2/3`(白/墨几个百分点),嵌套件是"染色"而不是"贴板"。

## 4. 纪律机制(防腐化的四道闸)

1. **辉光预算(Glow budget)**——白名单制,全站只有 7 处可以发光:`:focus-visible` 焦点环、导航激活态、
   live 圆点(唯一循环动画)、HUD 亮线、评分明细里的分段(仅暗色/仅全展开)、榜单色点/共鸣标记/开关 on、
   主按钮 hover。**正文永不发光**;亮色模式辉光变光晕(`--bloom: 0` 自动关闭所有 bloom,因为 bloom 都写成
   `color-mix(… var(--bloom) …)`)。
2. **对比度预算**——逐 token 标注:`--text-2/--text-3` 对 bg/surface/glass 全部 ≥4.5:1,`--accent` 10:1(暗)/5.2:1(亮),
   自定义 accent 由引擎按 WCAG 对比度**自动配墨色**(见 §5)。
3. **禁止清单**——玻璃无光照、动画渐变、整卡霓虹边框、正文发光、彩虹渐变、文字投影、emoji 装饰、
   循环装饰动画、**对玻璃容器做 opacity/filter/mask 淡出**(会切断 backdrop-filter 的折射场景,见 §6)。
4. **文档-代码一致性**——VISUAL.md 按条款编号(§1–§8),CSS 注释直接引用条款号(如 `/* VISUAL §5a */`),
   审查时可以像对法条一样逐条对账。

## 5. 主题引擎(L3)

- **html 属性即状态机**:`data-preset`(titanium|paper|terminal)× `data-mode`(light|dark)×
  `data-density`(compact|comfortable|cozy)× `data-motion`(reduce|full|缺省跟随系统)× `data-safe`。
- **级联(最后者胜)**:内置 CSS → `public/custom.css`(fork 的)→ Settings 自定义 CSS(`<style id="user-css">`,
  用 textContent 写入,永不解析成 HTML——防注入)→ token 内联覆盖 → **`?safe=1` 逃生舱**(URL 参数,
  一次性关掉 custom.css + 自定义 CSS + token 覆盖,砖了也能进)。
- **accent 色彩数学(纯函数)**:`parseColor`(hex/rgb/hsl 全解析)→ `luminance`(WCAG 相对亮度)→
  对比度比对 → 自动决定 `--accent-ink`(accent 上的文字用深还是浅)、hover 深化色、焦点环色。
  用户随手挑的颜色永远满足对比度——**把无障碍从"审查项"变成"运行时不变量"**。
- **boot.js 防闪白**:render-blocking 的 classic script(CSP 禁 inline,所以是外链),首帧渲染前把
  preset/mode/density/fontScale/accent/自定义 CSS 全部就位;与 prefs.ts 是镜像实现,
  `test/theme/boot.test.ts` 保证两者永不漂移。**这是所有带主题系统的站点都该抄的一段**。
- **settings 版本化**:localStorage 文档带版本号,改结构必须加 `MIGRATIONS[n]` + 测试;密钥永远不进
  settings(vault 单独存),settings 导出时可 `redact`。

## 6. 动效编排(motion choreography)

- 三档时长 `--dur-1 160ms / --dur-2 240ms / --dur-3 420ms`,两条曲线 `--ease`(抽屉曲线)/`--ease-pop`(旋钮回弹)。
- **上电动画**(620ms):面板从 98.5% 缩放 + 14px 上升 + 全息闪烁淡入,按 `--i` 以 70ms 错峰——
  hero → 简报 → 筛选器 → 各榜单网格序。
- **路径扫描线**(760ms):仅在**路径变化**时从顶栏下扫一条 signal 亮线;筛选和弹层不触发。
  入场动画只填 `backwards` 不填 `both`(定住的面板要能回到自己的 hover 样式)。
- **backdrop roots 规则**(整段都是坑的教训):祖先带 `opacity<1 / filter / mask / clip-path` 会把玻璃从
  折射场景上切下来,效果结束时玻璃会从透明"啪"地变磨砂。所以**永远逐面板自己淡,不淡容器**。
- hover 位移只在 `@media (hover: hover)` 下存在(触屏不飘);reduced-motion 下时长归零、指针光斑关闭、扫描线不出现。

## 7. 工程质量配套

- **测试**:theme boot 同步测试、i18n 中英 parity 测试、各源 HTML fixture 回放;532 文件近半是测试。
- **预算**:初始包 ≤95KB gzip;功能经 `core/registry.ts` 注册制接入(路由/设置页/条目动作/命令),
  重件(Anthropic SDK、tweetnacl)全部 lazy import。
- **CSP 严格模式**(禁 inline script)——所以 boot.js 是外链 classic script 而不是内联片段。
- **data-part 稳定钩子**:frame/today/items/states/features 五组命名,custom.css 可精确锚定组件而不碰类名。

## 8. 可移植性清单

| 类别 | 内容 |
|---|---|
| **直接搬** | 材质派生配方(§3);辉光预算/禁止清单(§4);boot.js 防闪白模式(§5);级联+`?safe` 逃生舱;VISUAL.md 契约范式;backdrop-roots 规则(§6);对比度逐 token 标注的习惯 |
| **改造搬** | 语义色层(换成自己产品的语义);自定义 accent 的 WCAG 色彩数学(engine.ts 纯函数可独立抽出);data-density;settings 版本化迁移 |
| **别搬** | 五榜单/八分类语义色(产品耦合);Preact registry(小站用不上);MCP/channels 投递(属产品不属设计系统);183 天趋势徽章(同左) |

---

# 第二部分 同类方案全景(2026-10-12 检索核实)

结论先行:**"token 化设计系统"已是成熟工程领域,分四大流派;AI-Resonance = CSS 原生派 + 文档契约派的混合,
且是个人项目里工程密度的天花板,但生态里没有任何一个方案同时做到它这五件事(派生材质/辉光预算/契约文档/
accent 色彩数学/安全模式)——最接近的单项对手分别列出如下。**

## 9. 流派 A:标准管线派(token 是数据,构建到多平台)

| 方案 | 星数 | 是什么 | 与 AI-Resonance 的异同 |
|---|---|---|---|
| **W3C DTCG Design Tokens Format** | — | 设计 token 交换格式的**官方规范**,**2025-10-28 定稿首个稳定版(2025.10)**,designtokens.org | AI-Resonance 的 token 是 CSS 变量原生形态,不参与 DTCG 交换;多团队/多平台协作时 DTCG 才是正道 |
| **Style Dictionary**(Amazon) | 4.9k★ | token 构建系统:一份 JSON → CSS/iOS/Android/Sketch 多端产物,W3C 规范参考实现 | 同上,构建派 vs 零构建派 |
| **Theo**(Salesforce) | 2.0k★ | token 变换/格式化引擎(先于 Style Dictionary) | 同上 |
| **Tokens Studio** | — | Figma 插件 ↔ 代码的 token 管理平台,W3C 参考实现 | 设计师工作流入口,AI-Resonance 没有设计稿环节 |
| **Terrazzo** | — | 新兴 token 构建工具,W3C 参考实现 | 同上 |

## 10. 流派 B:算法派(颜色是算出来的,对比度可证明)

| 方案 | 星数 | 是什么 | 与 AI-Resonance 的异同 |
|---|---|---|---|
| **Radix Colors** | 1.7k★ | 12 步语义刻度:**步=用途**(1-2 背景、6-8 边框、9-10 实底、11-12 可读文字),暗色模式刻度反转即得,双主题对比度自动达标 | 同样"语义刻度+自动达标"思想;Radix 是预制色板,AI-Resonance 是 3 预设×自定义 accent,且派生到材质不止颜色 |
| **Adobe Leonardo** | 2.2k★ | **目标对比比生色**:给定背景与目标对比度(3:1/4.5:1/7:1),CAM02 均匀空间反解颜色;Adobe Spectrum 的底座 | 与 engine.ts 的 accent 色彩数学同宗(WCAG luminance 反推 ink);Leonardo 是独立工具+JS 库,AI-Resonance 内嵌为运行时纯函数 |
| **Material 3 dynamic color**(material-color-utilities) | 2.3k★ | HCT 色彩空间(hue/chroma/tone),种子色→色调调色板→角色映射,Android 壁纸取色主题 | Google 的运行时主题引擎;AI-Resonance 的 color-mix 派生是它的"穷人版"(浏览器原生、零 JS 计算材质) |

## 11. 流派 C:CSS 原生派(token 就是 CSS 变量,零构建)——AI-Resonance 所属

| 方案 | 星数 | 是什么 | 与 AI-Resonance 的异同 |
|---|---|---|---|
| **Open Props**(Adam Argyle) | 5.5k★ | 现成 CSS 变量库(色阶/间距/缓动/阴影),CDN 即用 | 同为零构建 CSS 变量;Open Props 是**通用底料**,AI-Resonance 是**单品定制**+派生材质+契约文档,完成度高一个量级 |
| **Tailwind v4** | 97.9k★ | CSS-first 配置(`@theme`)、OKLCH、原生 color-mix | 同用现代 CSS;Tailwind 给的是原子类系统,AI-Resonance 给的是材质系统,两者可叠加 |
| **every-layout.dev / Utopia** | — | 布局原语 / 流式字号刻度 | 同流派的思想邻人,只覆盖布局/字号的单一维度 |

## 12. 流派 D:文档契约派(设计系统 = 一份 AI 与人都能读的契约)——AI-Resonance 的另一半

这是 2025–2026 增长最快的一派,**和 AI-Resonance 的 VISUAL.md 是同一物种**:

| 方案 | 热度 | 是什么 |
|---|---|---|
| **Google Stitch DESIGN.md** | 2026-04 官宣开源格式 | DESIGN.md 概念由 Google Stitch 提出:纯 Markdown 编码整套设计系统,给 AI agent 读(与 AGENTS.md 配对:AGENTS.md 管"怎么建",DESIGN.md 管"长什么样")。Stitch 内可跨项目导入导出 |
| **VoltAgent/awesome-design-md** | **120k★** | 73 份从真实品牌网站(Claude/Mistral/Runway/Stripe…)提取的 DESIGN.md 合集,"drop one into your project, tell your agent build me a page that looks like this" |
| **designmd.app** | 759+ 文件 | DESIGN.md 库,适配 Claude Code / Cursor 等 |
| **vibe-ui Design.md**(知识库来源 [1][3]) | — | 同思想中文先声:项目设计 DNA 固化成仓库文档 |
| **AI-Resonance docs/VISUAL.md** | 9★(但工程密度见第一部分) | 同范式更严:值=代码值、条款编号、CSS 注释引用条款、测试锁定 |
| **portal-v3 docs/VISUAL.md** | 我们的 | B2 晨报/晚报契约,2026-10-12 落地 |

## 13. 总对照表

| 方案 | token 形态 | 材质派生 | 主题机制 | 对比度保证 | 契约文档 | 适合谁 |
|---|---|---|---|---|---|---|
| AI-Resonance | CSS 变量 | ✅ color-mix 全派生 | 预设×模式×密度+boot | 运行时数学+逐 token 标注 | ✅ VISUAL.md 21KB | 个人/小团队 Web 单品 |
| Radix Colors | CSS 变量 | ❌(只管色) | 类切换 | 刻度内置 | ❌ | React 组件库配色 |
| Leonardo | JS 库/工具 | ❌ | 主题生成 | **对比比反解**(最强) | ❌ | 大厂多品牌主题 |
| Material 3 | 运行时引擎 | ❌ | 种子色→HCT→角色 | 色调映射 | ❌ | Android/全平台动态主题 |
| Open Props | CSS 变量 | ❌ | 亮暗两套 | 手工 | ❌ | 快速起步通用底料 |
| Style Dictionary+DTCG | JSON→多端 | ❌ | 构建期 | 无 | ❌ | 多平台设计团队 |
| Stitch DESIGN.md | Markdown 文档 | ❌ | — | 描述性 | ✅(本体就是文档) | AI 生成 UI 的约束入口 |

## 14. 结论

1. "设计系统当作软件工程"在 2026 年已四派并立:**标准管线派管交换,算法派管颜色正确性,CSS 原生派管零构建,
   文档契约派管 AI 协作**。四派不互斥,大厂通常 A+B,新潮个人项目 C+D。
2. AI-Resonance 是 C+D 混合的极端样本:9 star 的仓库做到了多数千星项目没做的三件事——
   **材质从 token 派生**(换色即换整套深度)、**辉光预算白名单**、**契约文档与代码同提交修复**。
3. 对我们(portal-v3/后续项目)的直接启示:**文档契约派已从社区实践升级为 Google 官方格式**
   (DESIGN.md 开源 + 120k★ 合集),我们刚落地的 docs/VISUAL.md 方向踩对了;
   下一步若做多主题,抄 AI-Resonance 的"预设只换值+材质派生";若做用户自定义 accent,抄 engine.ts 的 WCAG 反推。

---

# 第三部分 设计系统之外的可学工程模式(2026-10-12 补挖)

> 设计系统之外,这个仓库还有一批与视觉无关、但同样高密度的工程模式。
> 每条附一手证据与适用场景。

## 15. VERIFIED.md:证据台账(244KB)

外部事实全部登记成台账:每条标 `✔`(实测观察)或 `?`(仅来自文档,未实测),注明探测日期
("Probed live on 2026-09-19; re-verify before relying on anything marked ?")。内容细到:
GitHub trending 每页预期 15–25 行、search API 每 topic 单独查询(OR 限定符会 422)、
"OSSInsight/GH Archive/star-history 三个 star 历史源全部实测已废"、Reddit 403"network security"
意味着什么、X 各 provider 的价格与新鲜度……**记的不是结论,是证据+置信度+复验日期+反例**。
 scraping 配方逐条给正则与降级路径,失败模式当一等公民记录。

适用:长期个人项目的"坑台账"升级模板(替代只记结论的 DEV.md/踩坑笔记)。

## 16. HTTP 罔层:管道唯一的网络之门

`http.ts` 自述:"The pipeline's only door to the network"。一个文件收拢全部网络纪律:
- 超时 30s / 重试 2 次,退避 `1s·2ⁿ + jitter`,优先听 `Retry-After`;
- **每主机串行队列 + 最小间隔**(默认 200ms——"就算是没要求限流的主机,也别让 cron 突发打人");
- 读服务器自己的限流头(`x-ratelimit-remaining/reset`,按数值大小区分"秒数"与"epoch 秒"),预算耗尽才暂停,至多 60s;
- **fixture 录制/回放**:`RESONANCE_FIXTURES=record|replay`,`<host>-<hash>.json`,**请求头永不落盘(防 token 泄漏)、响应体里 access_token/refresh_token/id_token/token 字段自动替换为 REDACTED——录制样本就是设计来提交进仓库的**,全部上层阶段可离线测试;
- `HttpError` 保留响应体前 2000 字符,"够辨认这是一张封锁页";retryable 规则明确:429/5xx/403+Retry-After。
- 解析器自验证:trending 解析结果 <5 行或 >30% 空星标 ⇒ 判定解析器损坏 ⇒ 记健康标志、回退 search-only。**解析器不信任自己,先统计合理性再输出**。

适用:任何爬虫/API 客户端的第一块积木;比"裸 fetch + try/catch"高一个位面。

## 17. 存储降级链:永不抛错的 kv

`core/storage.ts`:IndexedDB → localStorage → memory 三级后端;**中途开始失败的后端被永久放弃**,
自动落到下一级;`get` 永不抛错(返回 undefined),`set` 失败返回 false(配额满/隐私模式),
所有后端统一存 JSON 字符串,行为处处一致。约 150 行,零依赖。

适用:前端一切"缓存型"存储(摘要缓存、搜索索引副本);写它的时间比调 IndexedDB 的坑少一个数量级。

## 18. 一份 DOM-free 渲染器,三端字节一致

`channels/src/report/render.ts`:同一个纯函数渲染器被三处消费——线上托管报告、前端 Export 视图、
邮件任务——**三处产物字节级一致**。报告设计原则:**无 JS 也完整**(邮件客户端预览、打印、
`file://` 禁脚本场景):卡片/简报/共振簇全部初始语言服务端渲染,另一语言副本就放在旁边用
CSS `data-l` 切换;数据以 JSON 内嵌 `<script type="application/json">`;内联控制器只加标签页/筛选/语言主题切换这些糖。

适用:portal-v3 若做"日报单文件存档"(某天存成离线 html)或邮件推送,这就是现成设计。
**渐进增强纪律:JS 是糖,不是骨头。**

## 19. 成本可观测:钱进数据结构

X 采集在 SourceStatus 里逐次上报 `mode` 与 `costUsd`;月度 USD 软上限记在 state;
provider 选择 `auto` = 按序取第一个配了 secret 的付费源,否则免费 syndication。
配额不是配置文件里的一个数字,而是**每次抓取都落盘的可观测状态**。

适用:按量付费 API(SenseFlow 转写、DeepSeek)的防账单惊吓:每次调用记成本,月上限熔断。

## 20. 分类透明化:matched_terms 落盘

AI 分类是规则打分:topic 命中=2 分、名称/描述正则命中=1 分(带词边界防 "maintain" 命中 "ai",
含中文词),≥2 收录;1–1.5 临界分只在 trending 出现时收录;人工黑/白名单 JSON 纠偏;
**matched_terms 随条目存下来,UI 的热度面板能解释"它为什么被算作 AI"**。
LLM 无权改分类("Categories come from a rule-based classifier and never change scores")。

适用:任何"为什么推给我/为什么命中"的溯源需求;与 zhiliao 规则引擎的命中统计同宗,可互相借鉴。

## 21. 仓库自带导读

`docs/READING-GUIDE.zh.md`(从哪读起)、`docs/LEARNING.md`(实施中学到了什么)、
`docs/IMPLEMENTATION-2026-09-19.md`(当日实施记录)。仓库不只说明"是什么",还说明"怎么读我"。

适用:长周期个人项目 + AI 接手场景(zcode-bridge 无头会话第一份喂给 agent 的文件就该是导读)。

## 22. 施工方法还原(git 史证据,2026-10-12 考古)

main 分支仅 **9 个提交、三波、23 天**,作者两种署名:人(W)3 个,AI(**Claude**)6 个:

| 日期 | 作者 | 内容 | 规模 |
|---|---|---|---|
| 09-19 09:18 | W | 首提交:**完整 v1 大爆炸**——schema/pipeline/web/channels/测试/CI/全套文档 | **445 文件** |
| 09-19 11:06 | W | 修复轮(按 review 文档,见 IMPLEMENTATION-2026-09-19.md,975 测试通过) | 487 文件(+42) |
| 09-21 09:16 | W | 新手推荐位 | ~489 |
| 09-25 03:25–12:01 | **Claude ×6** | **整个战术玻璃视觉层**(VISUAL/tokens/HUD/辉光)→ 90 分钟内连做两轮减法("calmer scene"/"declutter")→ 凭据页 → README 门面 | 532 文件 |

还原出的方法论(每条都有仓库物证):
1. **契约先行大爆炸**:首日 445 文件不是脚手架,是按 DESIGN.md 契约生成的完整产品;首提交信息是产品发布词不是 "init"。VERIFIED.md 同日实测落账。
2. **人是架构师+审查者,AI 是实施者**:AI 以自己的名字提交;人的提交是奠基波与合并,AI 的提交是完整功能纵切。
3. **审查驱动修复轮**:IMPLEMENTATION-2026-09-19.md 开头即"按 AI-Resonance-review-2026-09-19.md 的审查意见实现",结尾是验证回执(975 测试/tsc/Biome/build/真实 HTTP 回归/桌面+390px 手测无溢出无报错)。构建→独立审查→修复→验证→留档,闭环在同一天完成。
4. **测试是 AI 的缰绳**:975 项测试在第一天就位(schema 20/channels 134/pipeline 314/web 507)+ fixture 录制回放,大爆炸提交才敢做。
5. **文档取代式演进**:DESIGN.md"v2 条款取代下文不一致处",历史不清算,每波留带日期的实施回执。
6. **生成后立即减法**:视觉层 03:25 生成、04:29"calmer"、04:54"declutter"——探索与减法在同一晨完成,VISUAL.md 的辉光预算/禁止清单就是这次剪枝的成文残留。

诚实边界:本地是否 squash 过、生成会话内容、W 的提交里含多少 AI 代码,均不可见;但"契约+审查+测试回执"的方法论证据是充分的。
这正是本库 2.2/五节"约束派"路线的实证终点:人写规则与回执,AI 写代码,可复现。

## 附录:来源(均于 2026-10-12 核实)

- AI-Resonance 仓库与 VISUAL.md/tokens.css/engine.ts/boot.js 原文(GitHub API 一手抓取,存 `ai-resonance-VISUAL.md` 等)
- W3C DTCG:[designtokens.org](https://www.designtokens.org)(Design Tokens Format Module 2025.10,2025-10-28 首个稳定版)
- Style Dictionary:[github.com/style-dictionary/style-dictionary](https://github.com/style-dictionary/style-dictionary)(4.9k★)
- Theo:[github.com/salesforce-ux/theo](https://github.com/salesforce-ux/theo)(2.0k★) · DTCG 仓库 [design-tokens/community-group](https://github.com/design-tokens/community-group)(2.1k★)
- Radix Colors:[radix-ui.com/colors](https://www.radix-ui.com/colors) · [github.com/radix-ui/colors](https://github.com/radix-ui/colors)(1.7k★)
- Adobe Leonardo:[leonardocolor.io](https://leonardocolor.io) · [github.com/adobe/leonardo](https://github.com/adobe/leonardo)(2.2k★)
- Material 3:[m3.material.io](https://m3.material.io) · [material-color-utilities](https://github.com/material-foundation/material-color-utilities)(2.3k★)
- Open Props:[open-props.style](https://open-props.style) · [github.com/argyleink/open-props](https://github.com/argyleink/open-props)(5.5k★)
- Tailwind v4:[github.com/tailwindlabs/tailwindcss](https://github.com/tailwindlabs/tailwindcss)(97.9k★)
- Google Stitch DESIGN.md 开源官宣:[blog.google/models-and-research/google-labs](https://blog.google/models-and-research/google-labs)(2026-04-21)
- awesome-design-md:[github.com/VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md)(120.3k★,73 份品牌 DESIGN.md)
- DESIGN.md 库:[designmd.app](https://designmd.app)(759+ 文件)
