# AI 用量表 · Swiss Ledger 设计规格

版本：v2.0-draft  
视觉基准：`1Cash` 与 `Footy` 当前代码中的 Swiss Ledger 实现

## 1. 设计定位

AI 用量表不发明新的视觉语言。它继承 1Cash 与 Footy 已落地的 Swiss Ledger 体系，并把“财务账本 / 足迹档案”的语言转换成“AI 额度账本”。

一句话方向：

> 把分散在各平台的 AI 额度做成一张可随时核对的系统仪表账页，而不是彩色、玻璃感的监控面板。

气质关键词：Systematic、Precise、Restrained、Monochrome-first、Data-forward、Auditable、Instrument-like。

## 2. 继承规则

以下规则直接继承 1Cash / Footy，不允许实现 Agent 自行变体：

- Archivo 字体，CJK 自动回落 PingFang SC；
- 纸白 `#FAFAF7` / 近黑 `#0B0B0A`；
- 面板与页面同色，只靠发丝描边分层；
- 圆角统一 2 pt，徽标和进度条为直角；
- 无胶囊、无阴影、无玻璃材质；
- 数字使用等宽数字；
- 小节标题和状态标签使用全大写、加宽字距；
- 每个页面以黑白灰为主，只允许一个 App 专属强调色；
- 动效用于确认状态，不用于表演。

## 3. Swiss Ledger 强调色

AI 用量表继续使用 1Cash 已落地的 Swiss Ledger 印刷红，不另设专属色：

| Token | 浅色 | 深色 | 对比度 |
|---|---|---|---|
| `accent` | `#C4321F` | `#E2503A` | 纸底约 5.2:1；墨底约 5.1:1 |

选择原因：保持用户指定的 1Cash / Footy Swiss Ledger 视觉连续性，避免为了新 App 再创造一套品牌色。该红色只表示本产品当前最重要的单一信号，不代表危险；真正的危险状态使用更深的 `danger` token。

强调色只用于当前最紧张/选中的单一指标、主操作、菜单栏状态点、App 图标和当前活动连接器。额度危险仍使用 Swiss Ledger 的 `warning` / `danger`，但只用于小面积标签、刻线或图标，不把整张卡片染色。

## 4. Design Tokens

### 4.1 颜色

| Token | 浅色 | 深色 | 用途 |
|---|---|---|---|
| `pageBackground` | `#FAFAF7` | `#0B0B0A` | 页面与便签底色 |
| `panelBackground` | 同 page | 同 page | 仅靠描边区分 |
| `panelMutedBackground` | 黑 4% | 白 6% | 静音控件、弱选区 |
| `panelBorder` | 黑 14% | 白 14% | 面板发丝描边 |
| `divider` | 黑 10% | 白 10% | 行分隔 |
| `hairlineStrong` | 黑 16% | 白 16% | 强分隔 |
| `ink` | `#0B0B0A` | `#F5F4F0` | 主文本、主图形 |
| `inkSoft` | `#6E6E68` | 对应高对比灰 | 次要文本 |
| `inkFaint` | `#8A8981` | 对应 48% 灰 | 禁用和装饰 |
| `accent` | `#C4321F` | `#E2503A` | Swiss Ledger 印刷红 |
| `success` | `#2F7A63` | `#3F9E82` | 正常/恢复 |
| `warning` | `#8A6100` | `#C9962A` | 额度偏低/过期 |
| `danger` | `#9E1B12` | `#F2684F` | 耗尽/不可逆操作 |

单个 surface 同时出现的有彩色元素不超过两种；通常为 accent 加一个必要状态色。

### 4.2 字体

沿用 1Cash / Footy 的 `LedgerType`：Archivo Regular、Medium、SemiBold、Bold、ExtraBold。

| Role | 规格 | 用途 |
|---|---|---|
| `hero` | 38 pt ExtraBold | 详情中的主要剩余量 |
| `pageTitle` | 32 pt ExtraBold | 设置/详情标题 |
| `meterValue` | 24 pt ExtraBold | Popover 主百分比 |
| `rowValue` | 13.5 pt Bold | 平台行数值 |
| `sectionLabel` | 11 pt Bold + tracking 1.2 | 分组标题 |
| `body` | 17 pt Regular | 标准正文 |
| `rowTitle` | 13.5 pt SemiBold | 平台名称 |
| `rowMeta` | 10.5–11 pt Regular | 重置、更新时间 |
| `badge` | 9 pt Bold + tracking 0.6 | 状态标签 |
| `button` | 15 pt Bold | 主按钮 |

所有 Token、积分、金额、百分比和时间倒计时必须 `.monospacedDigit()`。

### 4.3 间距

```text
pageHorizontal    18
cardPadding       16
cardPaddingLarge  20
sectionSpacing    12
formSpacing       16
itemSpacing        8
rowIconSpacing    12
pageBottom        24
textStackSpacing   3
captionSpacing     2
segmentSpacing     4
compactSpacing     6
```

Popover 因面积更小可使用 12 pt 外边距，但组件内部仍使用上述 token，不新增随意档位。

### 4.4 形状

```text
panelCornerRadius    2
controlCornerRadius  2
badgeCornerRadius    0
progressBarRadius    0
iconBoxSize         34
borderWidth          1
```

禁止 Capsule、10 pt 以上圆角、连续圆角大卡和彩色圆形 Provider 图标底座。

### 4.5 动效

- 标准切换：0.18 秒 easeInOut；
- 退出：0.18 秒 easeOut；
- 数值更新：轻微 crossfade 或 content transition；
- 禁止呼吸灯、持续闪烁、弹簧式进度条；
- Reduce Motion 下全部降级为即时切换或淡入淡出。

## 5. 图形语言

### 5.1 额度表达

不使用圆环作为默认主组件。Swiss Ledger 使用直角“刻度线”：

```text
QODER WEEKLY                              18% LEFT
██████████████████░░░░░░░░░░░░░░░░░░░░░░
RESET 02H 14M · OFFICIAL QUOTA
```

- 轨道高 2–4 pt；
- 默认填充 `ink`；
- 当前选中的关键额度可用 `accent`；
- 警告/危险时仅填充条和状态标签变色；
- 未知值显示空刻线与 `UNAVAILABLE`，不是 0%。

### 5.2 Provider 标记

- 使用 8×8 pt 方形 mark；
- 未选中：描边；
- 有本地活动：实心 `ink`；
- 当前关键项：实心 `accent`；
- Provider 官方图标只可在详情或连接器设置中低调出现，不主导主界面颜色。

### 5.3 状态标签

语法：全大写、9 pt、加宽字距、1 pt 描边、不填色。

| 标签 | 含义 | Tone |
|---|---|---|
| `LIVE` | fresh、正常 | neutral/success |
| `LOCAL` | 本地结构化数据 | neutral |
| `API` | 平台接口数据 | accent |
| `EST.` | 估算 | neutral |
| `STALE` | 数据过期 | warning |
| `LIMIT` | 接近额度 | warning |
| `EXHAUSTED` | 已耗尽 | danger |
| `AUTH` | 需要授权 | warning |
| `BETA` | 实验连接器 | neutral |
| `ERROR` | 解析/连接失败 | danger |

## 6. 菜单栏标题

菜单栏本身遵循 macOS 可读性，不强行渲染 Archivo 图形。格式：

- 自动：`▰ 18%`
- 双指标：`Q 18% · C 62%`
- 极简：App 单色图标 + 一个方形状态点

标题最大宽度 160 pt。默认语义为剩余，若来源只能提供已用，必须显示 `USED`。

## 7. Popover

尺寸：360×自适应，最大高度 560 pt。背景使用 Swiss Ledger 纸白/墨黑 surface；窗口可保留系统阴影，内容层不得叠加阴影或玻璃材质。

```text
┌────────────────────────────────────────┐
│ [AM] AI METER            LIVE  10:42 ↻ │
├────────────────────────────────────────┤
│ USAGE LEDGER · 06 SOURCES               │
│                                        │
│ QODER WEEKLY                   18% LEFT │
│ ███████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ RESET 02H 14M                   [API]   │
├────────────────────────────────────────┤
│ □  CODEX       62%     RESET THU 18:00 │
│ ■  WORKBUDDY   1,240   CREDITS         │
│ □  QWEN WORK   74%     MONTHLY         │
├────────────────────────────────────────┤
│ FLOATING LEDGER  OFF     SETTINGS  QUIT │
└────────────────────────────────────────┘
```

### 7.1 顶部系统栏

- 高 42–48 pt；
- 左侧 22×22 pt 方形 monogram `AM`；
- 产品名使用 13 pt ExtraBold；
- 右侧状态标签、时间和方形刷新按钮；
- 底部 1 pt divider。

### 7.2 关键额度区

- 只显示一个最紧张额度；
- section label 全大写；
- 24 pt 等宽主数字；
- 2–4 pt 直角刻度线；
- 不放大面积彩色背景。

### 7.3 Provider 记录行

- 高 48–56 pt；
- 左侧方形 mark；
- 名称左对齐；
- 主数值与 reset 右对齐；
- 行间 1 pt divider；
- 不把每行包成独立圆角卡片。

### 7.4 Footer

- 上边框 1 pt hairlineStrong；
- 全大写小字；
- 开关使用 Swiss Ledger 方形 `AppSwitchStyle`，不使用系统胶囊 Toggle。

## 8. 桌面悬浮便签

桌面便签是一张“浮动账页”，不是毛玻璃组件。

### Small：220×116 pt

```text
┌────────────────────────────┐
│ AI METER        LIVE 10:42 │
├────────────────────────────┤
│ QODER       18%      02H14 │
│ CODEX       62%         THU│
│ WORKBUDDY 1,240      CREDIT│
└────────────────────────────┘
```

- 2 pt 外角；
- 纸白/墨黑实底；
- 1 pt 外框；
- 无内容阴影；
- 三行上限；
- 数值右对齐。

### Medium：300×220 pt

- 顶部反色 ledger band：`ink` 实底 + `pageBackground` 文字；
- 一个 hero 额度；
- 最多 5 个次级记录行；
- 底部更新时间和刷新；
- 不画图表、不画圆环。

### 窗口行为

- 默认 92% 不透明度；最低 75%，避免破坏账页质感；
- 默认普通浮动层级，不覆盖全屏；
- 标题区拖动；
- 鼠标离开时隐藏操作按钮；
- `⌘⇧U` 切换显示；
- 位置按屏幕 UUID 记忆。

## 9. 轻量详情

详情沿用 1Cash 的 Ledger Header Block：

- 顶部反色 band：平台名、账号、连接状态；
- Hero：主要剩余量；
- 三个并排小统计：今日、7 日、重置；
- 直角刻度线；
- 下方是记录行，而不是卡片网格；
- 唯一趋势图使用直角累计柱或刻度条，不使用渐变面积图和甜甜圈图。

## 10. 设置与连接器中心

采用标准 macOS 设置窗口，但内容仍遵循 Swiss Ledger：`SystemPageHeader`、`SystemPanel`、`SystemSectionLabel`、`SystemStatusBadge`、`StandardRow`、`AppSwitchStyle`、`AppErrorBanner` 和 `AppEmptyStateView`。

连接器行：左侧 34×34 pt 方形图标框，中间名称/来源，右侧状态标签和 chevron。不得使用彩色 Provider 卡片墙。

## 11. 图标

App 图标继承家族语言：

- 恒定近黑 `#0B0B0A` 背景；
- 单一 `accent` 实心图形；
- 无渐变、阴影、描边和文字；
- 图形建议为三条不同高度的直角仪表刻度 + 一条基准线；
- 避免速度表、机器人、Token 硬币、闪电和供应商 Logo。

菜单栏 template image 为纯单色：三条直角用量刻度。

## 12. 文案语气

Swiss Ledger 文案短、事实化、可核对。

推荐：`18% 剩余`、`02H 14M 后重置`、`数据更新于 8 分钟前`、`平台未提供 Token 明细`、`API 等效费用 · 非实际账单`、`连接器需要重新授权`。

避免 emoji、游戏化庆祝、夸张告警、口语化“烧 Token”，以及把未知显示为 0。

## 13. 无障碍

- Archivo 缺少 CJK 时必须可靠回落；
- 主文本至少达到 4.5:1；
- `inkFaint` 不用于小号正文；
- 状态由颜色 + 标签 + 图标共同表达；
- 交互区域至少 28×28 pt，关键按钮 34×34 pt；
- VoiceOver 朗读完整语义；
- Increased Contrast 下加深 hairline；
- Reduce Transparency 下保持实底，不影响布局；
- Reduce Motion 下取消数值滚动。

## 14. 禁忌

- 圆环仪表盘作为默认视觉；
- 毛玻璃、Liquid Glass 和大面积透明模糊；
- 10–20 pt 圆角卡片；
- 胶囊 Chip/Toggle；
- 彩色 Provider Logo 墙；
- 渐变进度条；
- 阴影堆叠；
- 每个平台一种品牌色；
- 饼图、甜甜圈图和装饰性波形；
- AI 星星、机器人、闪电、硬币等陈词滥调；
- 弹跳、呼吸、持续闪烁。

## 15. 设计组件复用

实现 Agent 应从 1Cash / Footy 的真实组件语法复刻：`LedgerType`、`FamilyTypography`、`FamilyUI`、`AppSpacing`、`AppCornerRadius`、`AppStroke`、`AppMotion`、`SystemPanel`、`SystemPageHeader`、`SystemPanelDivider`、`SystemStatusBadge`、Ledger Buttons、`AppSwitchStyle` 和 `StandardRow`。

不得直接让 AIUsageMeter target 依赖 1Cash 或 Footy target；应抽取/复刻共享 token，并在本工程增加契约测试。

## 16. Token 回归测试

必须建立 `DesignSystemTokenTests`，至少断言：

- 所有面板/按钮/图标圆角均为 2；
- badge/progress radius 为 0；
- 间距与家族 token 一致；
- Archivo 五个字重随包分发并可解析；
- accent 浅/深值固定；
- 所有数值组件启用 monospaced digit；
- 工程中不存在 `Capsule()`、内容 `.shadow(`、10 pt 以上 card corner；
- Provider 行不使用供应商品牌色作为背景。

## 17. 设计验收

- 与 1Cash Ledger 和 Footy 的组件并排时，能一眼看出属于同一家族；
- Popover 首屏不出现圆角卡片网格；
- Small/Medium 便签像账页而非系统玻璃组件；
- 1、3、6、12 个 Provider 时信息层级稳定；
- 0.5%、9%、100%、1,240 credits、unknown 均不破版；
- `LIVE/STALE/EST./LIMIT/ERROR` 无色环境下仍可区分；
- 浅色、深色、高对比和 Reduce Transparency 均可读；
- 实机截图需与 1Cash/Footy Swiss Ledger 参考并排验收。
