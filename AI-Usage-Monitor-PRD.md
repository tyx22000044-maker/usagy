# AI 用量表（暂定名）PRD

> 文档版本：v0.2  
> 调研日期：2026-09-20  
> 目标平台：macOS  
> 产品阶段：立项 / 技术验证前  
> 一句话定位：常驻 Mac 菜单栏或桌面的轻量 AI 用量表，一眼看清国内外 AI 工具还剩多少、何时重置。

---

## 1. 执行摘要

这个方向有真实需求，但“支持更多平台的菜单栏计数器”本身已经不是空白市场。

GitHub 调研显示：

- [usageBar](https://github.com/ChanningYuan/usageBar) 已经原生覆盖 Qoder CLI / Work / IDE、千问办公、WorkBuddy，以及 Claude Code、Codex、Cursor、OpenCode 等工具，并处理了累计快照差分、日志删除后的持久账本、远程价格表等问题。
- [TokenTracker](https://github.com/xiufengsun/TokenTracker) 已覆盖 39 类工具，明确包含 CodeBuddy、WorkBuddy、Qoder、Qoder CN、TRAE Work CN，并同时提供桌面端、组件、宠物和成就系统。
- [tokscale](https://github.com/junhoyeo/tokscale) 已覆盖数十种客户端，包含 TRAE 国际版、CodeBuddy、WorkBuddy、Qwen、Kimi 等；其中 TRAE 国际版通过已有登录态调用会话级 usage API。
- [OpenUsage](https://github.com/robinebers/openusage)、[AI Usage](https://github.com/burakgon/ai-usage-menubar)、[MeterUsage](https://github.com/pekth/meterusage) 已经把“原生 Mac 菜单栏 + 额度窗口 + 重置时间 + 低资源占用”做到了较高完成度。
- [CC Switch](https://github.com/farion1231/cc-switch) 的核心仍是多 CLI 配置、供应商切换、代理与技能管理，用量查询是其能力之一；它支持官方订阅、API 余额和自定义查询脚本，但不等于完整覆盖国内封闭式桌面客户端。

因此，本产品不是重型分析后台，而应是一款“抬眼就能看懂”的系统小工具。建议采用以下差异化：

1. **中国平台优先，但不止于中国平台**：以 Qoder、TRAE、CodeBuddy、WorkBuddy、千问办公等为首发优势，同时保留 Claude、Codex、Cursor、OpenCode 等全球基线。
2. **不只展示数字，还解释数字**：任何用量都必须显示来源、口径、置信度、更新时间和是否估算。
3. **菜单栏与桌面便签优先**：主界面保持在一张小卡片内，分析和设置只作为二级入口。
4. **同时管理额度、Token、积分和真实/等效成本**：避免把订阅额度、API 费用、平台积分和 API 等效价格混成一个“花费”。
5. **把“快用完了”变成可行动建议**：提供耗尽预测、异常消耗、重置提醒，以及有余量的平台/模型建议。
6. **把适配器维护能力做成产品护城河**：不同工具频繁变更日志、数据库和内部接口，必须建立版本探测、契约测试、诊断包和快速修复机制。

产品不应以“全平台一次性覆盖”为 MVP 目标。正确路径是：先用 6–8 个高价值且可验证的平台建立可信数据引擎，再扩展覆盖面。

---

## 2. 背景与问题

用户通常同时订阅或使用多个 AI 工具：Claude Code、Codex、Cursor、OpenCode，以及 Qoder、TRAE、CodeBuddy、WorkBuddy、千问办公等国内产品。这些工具的计量方式差异很大：

- Token：输入、输出、缓存读取、缓存写入、推理 Token；
- 额度窗口：5 小时、每日、每周、每月、按模型独立额度；
- 积分：不同模型有不同倍率，积分不能直接等同于 Token 或人民币；
- 费用：真实账单、套餐内消耗、API 等效成本、平台返回的美元成本；
- 请求次数：有些平台只暴露请求数或高级请求数；
- 数据来源：本地 JSONL、SQLite、Hook、已登录凭证、官方 API、未公开接口或网页账单。

现有工具的常见问题：

- 只覆盖海外主流工具，或只覆盖 CLI；
- 覆盖平台很多，但数字来源和准确度不透明；
- 把客户端、模型供应商和模型混为一谈，例如“在 Qoder 中使用 Claude”无法正确归因；
- 只能看已使用量，不能回答“按当前速度还能用多久”；
- 适配器失效时只显示空白或 0，用户无法判断是没有使用还是采集失败；
- 读取本地凭证或调用内部接口时，缺少明确授权和安全说明；
- 本地日志被清理、压缩或重写后，历史数字突然下降。

---

## 3. 产品目标与非目标

### 3.1 产品目标

1. 在一个 Mac 菜单栏入口中查看所有已连接 AI 工具的当前额度、今日消耗和重置时间。
2. 在不离开当前工作的情况下，通过菜单栏或桌面小卡片查看最重要的 3–8 个指标。
3. 每个数字都可追溯到来源，并明确标记“平台返回 / 本地记录 / 估算”。
4. 在额度耗尽、异常激增、采集失效和即将重置时主动提醒。
5. 通过稳定的适配器协议快速增加和修复国内外平台。
6. 默认本地优先，不采集提示词、回复正文、源代码和文件内容。

### 3.2 非目标

MVP 不做：

- 不做代理或中间人，不接管模型请求流量；
- 不做供应商/API Key 切换，避免与 CC Switch 正面重叠；
- 不承诺所有平台都有“实时 Token”；平台不提供时只展示可验证的积分、额度或请求数；
- 不把 API 等效价格宣传为真实账单；
- 不读取或分析提示词、回复内容和代码正文；
- 不在 MVP 提供企业级云端管理后台；
- 不做需要长期停留浏览的重型 Dashboard；
- 不把项目管理、会话管理、模型聊天或供应商切换塞进小工具；
- 不通过自动化破解、注入或绕过系统安全机制获取数据。

---

## 4. 目标用户

### 4.1 核心用户：多工具重度开发者

- 同时使用 3 个以上 AI 编程工具；
- 在额度不足时会切换平台或模型；
- 关心 5 小时/每周额度、Token 构成和 API 等效成本；
- 愿意授予本地文件只读权限，但对凭证外泄非常敏感。

核心任务：

> 我想在开始一段工作前知道哪个工具还有额度，并在被限流前收到提醒。

### 4.2 次核心用户：订阅成本管理者

- 同时购买多个个人订阅；
- 想判断哪个订阅使用不足、哪个经常耗尽；
- 需要按月查看“订阅费、实际使用、API 等效价值”。

核心任务：

> 我想知道哪些订阅值得续费，以及真实使用是否匹配月费。

### 4.3 扩展用户：小团队负责人

- 需要汇总成员自行选择的 AI 工具；
- 关注成本、工具采用率和异常消耗；
- 不能接受上传提示词或代码。

团队功能放在 v2，不进入首发 MVP。

---

## 5. 市场与 GitHub 竞品调研

### 5.1 竞品矩阵

| 项目 | 形态 | 主要覆盖 | 优势 | 主要空位 |
|---|---|---|---|---|
| [usageBar](https://github.com/ChanningYuan/usageBar) | 原生 macOS 菜单栏 | Claude、Codex、Cursor、OpenCode、Qoder 全家桶、千问办公、WorkBuddy 等 | 国内平台直读、本地优先、持久账本、Qoder 覆盖深 | 国内更多封闭平台、适配器生态、用户可见的数据可信度体系仍有空间 |
| [TokenTracker](https://github.com/xiufengsun/TokenTracker) | CLI + Web Dashboard + 原生桌面/组件 | 39 类工具，含 CodeBuddy、WorkBuddy、Qoder、TRAE Work CN | 覆盖广、Hook 与被动扫描并用、跨平台、游戏化 | 产品信息密度高；“额度决策”不是唯一核心；Mac 原生轻量体验可继续差异化 |
| [tokscale](https://github.com/junhoyeo/tokscale) | CLI / TUI / Web | 数十种编码客户端，含 TRAE、CodeBuddy、WorkBuddy | 适配器覆盖最广之一、会话/模型/工作区维度强、TRAE 国际版同步 | 不是常驻原生 Mac 产品；额度提醒和日常决策体验不是主战场 |
| [OpenUsage.ai](https://github.com/robinebers/openusage) | 原生 macOS 菜单栏 | Claude、Codex、Cursor、Copilot、Devin、Grok 等 | 原生体验成熟、模块化 ProviderRuntime、签名公证与自动更新 | 国内封闭式客户端覆盖弱 |
| [OpenUsage.sh](https://github.com/janekbaraniewski/openusage) | 跨平台 TUI / Daemon | 36 类 Agent、API 平台和本地模型 | Burn rate、报表、Prometheus、深度分析 | 对普通 Mac 用户不够“零学习成本”；国内桌面平台不是重点 |
| [CC Switch](https://github.com/farion1231/cc-switch) | Tauri 桌面端 | Claude、Codex、Gemini、OpenCode、OpenClaw 与多 API 供应商 | 配置切换、代理、失败转移、自定义额度查询强 | 是“配置与供应商管理器”，不是国内 AI 客户端统一计量器 |
| [AI Usage](https://github.com/burakgon/ai-usage-menubar) | 原生 macOS 菜单栏 | Claude、Codex、Cursor、Copilot、Antigravity、Devin、Grok | 极轻量、只看真实额度、低后台开销 | 不做历史扫描，不提供长期趋势和国内工具 |
| [Coding Usage Bar](https://github.com/hanzhangzzz/coding-usage-bar) | SwiftBar / CLI | Claude、Codex、GLM、DeepSeek、MiniMax、Kimi | “消耗速度是否合理”的 pacing 判断 | 产品形态偏技术用户、客户端覆盖有限 |

### 5.2 关键实现发现

#### Qoder

- Qoder CLI 可通过本地 transcript 获取四类 Token；官方文档也提供用量/额度查看能力。
- Qoder IDE 可从本地 `SharedClientCache/.../local.db` 的消息 Token 字段读取。
- Qoder Work 可能需要从应用日志增量镜像。
- Qoder CN 应作为独立客户端/账号域处理，不能默认与国际版合并。
- 官方团队 API 支持按 IDE、CLI、Web、QoderWork 等来源返回 usage event，未来可用于团队版。

结论：**高可行，适合 P0；必须把 CLI、IDE、Work 分别采集，再在 UI 中允许合并。**

#### WorkBuddy

- 已有项目证明可从 `~/.workbuddy/projects/**/*.jsonl` 读取 Claude Code 风格会话。
- 另有本地数据库/资源信息可提供 session usage、credit 汇总或账户额度，但不同版本路径和字段可能变化。

结论：**高可行，适合 P0；同时保留“Token 事件”和“平台积分”两个口径。**

#### CodeBuddy

- 已有项目通过 `~/.codebuddy/settings.json` 注入 SessionEnd Hook 获取使用信息。
- CodeBuddy 桌面端存在本地登录态；第三方项目可读取账户信息，但精确剩余额度接口并不稳定或未公开。
- CodeBuddy 官方以积分计量，不同模型/任务复杂度可能有不同积分倍率。

结论：**Token/积分事件可做，准确剩余额度需技术验证；MVP 可先显示已观测消耗，不能伪造剩余值。**

#### TRAE

- TRAE 国际版存在会话级 usage API，可从桌面端已有登录态中取得 JWT 后按会话同步，服务端还可能返回成本字段。
- TRAE 中国版、TRAE SOLO CN、TRAE Work CN 的能力并不一致。开源项目之间也存在“无会话级 API”与“可通过内部 API opt-in 读取”的版本差异，说明契约高度不稳定。
- 本地 Chat 数据库不等于本地 Token 真值；部分数据库还可能加密。

结论：**国际版可做 P1；中国版先做实验性连接器和版本探测，不作为首发准确性承诺。**

#### 千问办公 / Qwen Work

- 现有开源实现表明可从本地 segment 获取请求 Token，并可选同步网页账单/积分历史。
- 本地 Token 和账单积分是不同事实，需要分别保存、分别展示。

结论：**适合 P0，但联网账单必须显式授权。**

#### 百度 Comate、CodeGeeX 等 IDE 插件

- 可以可靠检测安装、配置和技能调用，不代表能可靠取得 Token 或剩余额度。
- 可能需要读取 VS Code/JetBrains 插件的本地数据库、日志或调用其账户接口；必须逐版本验证。

结论：**先进入“研究中”列表，不应为了宣传覆盖而显示不可信数字。**

### 5.3 竞品结论

“平台数量”很容易被复制，也已经有人做得很广。真正可形成壁垒的是：

- 数据口径正确；
- 同一账号跨产品线去重；
- 适配器失效时快速发现和修复；
- 用户能理解每一个数字；
- 从统计升级到预算与切换决策；
- 在国内封闭平台上持续维护版本兼容。

---

## 6. 产品定位

### 6.1 定位语

> 你的 AI 额度表：常驻 Mac 菜单栏或桌面，一眼看清还能用多少、什么时候耗尽、什么时候重置。

### 6.2 核心价值

1. **一眼知道还能用多少**：额度、积分、剩余时间和预计耗尽时间。
2. **知道数字从哪里来**：每个指标都有来源、置信度和口径。
3. **知道为什么消耗快**：按客户端、模型、项目、会话和 Token 类型拆分。
4. **知道下一步怎么做**：提醒用户减速、等待重置或切换到有余量的工具。

### 6.3 差异化原则

- 中国平台优先；
- 可信度优先于覆盖数量；
- 剩余额度优先于累计炫耀；
- 一眼可读优先于图表数量；
- 常驻小卡片优先于完整 Dashboard；
- 可行动建议优先于复杂报表；
- 本地与隐私优先于云端账户体系。

---

## 7. 数据定义与统一口径

这是产品最重要的基础设计。

### 7.1 五层实体模型

| 层级 | 示例 | 说明 |
|---|---|---|
| Client / 客户端 | Qoder IDE、TRAE、WorkBuddy、Codex CLI | 用户实际使用的应用或工具 |
| Account / 账号 | Qoder CN 个人账号、OpenAI Team 账号 | 同一客户端可有多个账号 |
| Plan / 套餐 | Pro、Max、月度积分包 | 定义额度窗口和计费规则 |
| Provider / 模型供应商 | Alibaba、OpenAI、Anthropic、DeepSeek | 与客户端品牌分开 |
| Model / 模型 | Qwen、GLM、DeepSeek、Kimi、Claude Sonnet | 同一客户端中可能调用多个模型 |

例如：“用户在 CodeBuddy 中调用 Claude Sonnet”必须记录为：

- Client = CodeBuddy
- Provider = Anthropic
- Model = Claude Sonnet
- Account/Plan = 用户的 CodeBuddy 账号与套餐

### 7.2 指标类型

#### Usage Event

- inputTokens
- outputTokens
- cacheReadTokens
- cacheWriteTokens
- reasoningTokens
- requestCount
- creditsUsed
- duration
- modelId
- projectIdHash
- sessionIdHash
- occurredAt

#### Quota Snapshot

- quotaName
- total / used / remaining
- unit：token、credit、request、CNY、USD、percent
- windowStart / windowEnd / resetAt
- capturedAt

#### Cost Event

必须区分：

- `billed`：平台或账单返回的真实费用；
- `subscription_allocated`：订阅费按时间或使用量分摊；
- `api_equivalent_estimate`：按公开 API 单价估算；
- `vendor_reported_estimate`：平台返回但非正式账单的金额。

四类费用不得在默认总额中混加。

### 7.3 数据来源与置信度

每个数字必须带来源标签：

| 等级 | 来源 | UI 标签 | 示例 |
|---|---|---|---|
| A | 官方账单/官方额度 API | 平台数据 | 账户积分余额、官方 5h 窗口 |
| B | 本地结构化 usage 字段 | 本地记录 | JSONL/SQLite 中的 Token 字段 |
| C | Hook 捕获的结构化事件 | 本地 Hook | SessionEnd 返回的 usage |
| D | 基于本地事件和价格表计算 | 估算 | API 等效成本 |
| E | 不完整字段推算 | 低置信估算 | 只有 total token，无输入输出拆分 |

UI 中禁止把 C–E 级数据伪装成“官方账单”。

### 7.4 去重原则

- 优先使用 requestId / messageId / sessionId；
- 累计快照必须先差分，不能直接求和；
- 恢复会话、压缩会话、重复导入不得重复计数；
- 本地日志和服务端账单可能描述同一请求，但默认不互相相加；
- 同一账号跨 Qoder CLI / IDE / Work 的账户总额度，只保留一个账户快照；客户端事件仍分别统计；
- 保存源文件指纹、逻辑事件键和 parserVersion，支持重算与迁移。

---

## 8. MVP 范围

### 8.1 首发连接器建议

P0 连接器：

1. Claude Code：建立成熟基线与测试口径；
2. Codex：验证累计快照差分和额度窗口；
3. Qoder CLI；
4. Qoder IDE；
5. Qoder Work；
6. WorkBuddy；
7. CodeBuddy；
8. 千问办公 / Qwen Work。

P1（Beta 后 1–2 个版本）：

- TRAE 国际版；
- TRAE Work CN（实验性）；
- Cursor；
- OpenCode；
- Kimi Code / Kimi Work；
- GLM / Z.ai Coding Plan；
- DeepSeek、MiniMax、Moonshot API 余额。

研究池：

- 百度 Comate；
- CodeGeeX；
- 豆包相关工作台/编程产品；
- 其他只提供网页端用量、没有稳定本地记录或 API 的平台。

### 8.2 菜单栏

默认显示：

- 当前最紧张的额度；
- 剩余百分比；
- 预计耗尽时间或重置倒计时；
- 状态颜色：正常、注意、危险、数据过期。

点击展开：

- 所有已连接工具的卡片；
- 今日 Token/积分/请求；
- 当前额度窗口；
- 重置时间；
- 最新同步时间；
- 数据来源徽标；
- 快速进入详情、立即刷新和诊断。

菜单栏支持三种紧凑程度：

- 极简：只显示最紧张额度，例如 `Qoder 18%`；
- 标准：显示两个重点指标，例如 `Qoder 18% · Codex 62%`；
- 自动：轮播或动态选择预计最先耗尽的 1–2 项。

### 8.3 桌面便签

桌面卡片是菜单栏的第二种呈现方式，不是另一套产品。

- 可固定在桌面或所有空间；
- 支持小、中两种尺寸；
- 小尺寸显示 1–3 条 Swiss Ledger 直角额度刻度和重置倒计时；
- 中尺寸显示 3–6 个平台、今日消耗和最近提醒；
- 支持调节透明度、紧凑度、是否始终置顶；
- 点击平台行展开轻量详情，不跳转到复杂主窗口；
- 数据过期时明确显示灰色和最后更新时间；
- 可选择仅使用菜单栏、仅使用桌面便签，或两者同时启用。

产品实现上应区分：

- **系统桌面组件（WidgetKit）**：更符合 macOS，但刷新受系统调度限制，适合分钟级额度展示；
- **悬浮便签窗口**：可接近实时刷新、交互更灵活，但需要处理置顶、空间切换和窗口打扰问题。

MVP 建议先做“菜单栏 + 可选悬浮便签”，WidgetKit 组件放到首发后续版本。

### 8.4 轻量详情页

- 从卡片点击后以小型 Popover 或 Sheet 打开；
- 显示今天、近 7 日和当前账单周期三个时间范围；
- 显示按客户端与模型的简单拆分；
- 最多一张趋势图，不做多层分析工作台；
- 积分、请求和 Token 分开展示，不强制换算；
- 真实费用与 API 等效估算分开展示；
- 提供“这个数字怎么算的”和连接诊断入口。

### 8.5 连接器详情页

- 安装检测结果；
- 账号与套餐；
- 可用能力：Token / 积分 / 额度 / 账单 / 项目归因；
- 数据路径或 API 域名；
- 当前权限；
- 最近成功与失败时间；
- parser/contract 版本；
- 数字解释：“该数字如何得到”；
- 测试连接；
- 重新扫描；
- 导出匿名诊断包。

### 8.6 提醒

MVP 支持：

- 剩余 20%、10%、5%；
- 按当前速度预计 30 分钟内耗尽；
- 今日消耗显著高于过去 7 日同时间段；
- 距离重置 10 分钟；
- 数据源超过设定时间未更新；
- 连接器解析失败或登录过期。

提醒必须可按平台、指标和时间段独立关闭。

### 8.7 可解释性

每个指标点击后显示：

- 数字：当前值；
- 含义：已用还是剩余；
- 口径：Token、积分、请求或金额；
- 范围：客户端、账号、模型或窗口；
- 来源：本地文件 / Hook / API；
- 质量：官方、本地记录或估算；
- 时间：数据生成时间与最后同步时间；
- 限制：已知缺失项。

---

## 9. 关键用户流程

### 9.1 首次启动

1. 展示隐私承诺：不读取提示词、回复正文和代码。
2. 自动检测已安装的工具。
3. 列出可直接读取、需要文件授权、需要联网授权的连接器。
4. 用户逐项开启。
5. 首次扫描并显示每个连接器的数据质量。
6. 60 秒内在菜单栏或桌面便签看到有效数据。

### 9.2 日常查看

1. 用户查看菜单栏最紧张额度。
2. 展开查看每个平台剩余量和重置时间。
3. 点击异常平台，查看客户端和模型的简单消耗拆分。
4. 根据可用额度建议切换工具，或等待重置。

### 9.3 数字有争议

1. 点击数字旁的信息图标。
2. 查看来源、时间、置信度和计算方式。
3. 与平台页面对比。
4. 执行重新同步或打开诊断。
5. 导出不包含凭证、路径和内容的诊断包。

---

## 10. 信息架构

```text
菜单栏 Popover
├── 最紧张额度
├── Provider 卡片列表
├── 今日消耗
├── 提醒 / 连接异常
└── 打开轻量详情

桌面便签
├── 小尺寸
├── 中尺寸
└── 置顶 / 透明度 / 空间

二级界面
├── 轻量详情
│   ├── 今天 / 7日 / 账单周期
│   ├── 一张趋势图
│   └── 数字解释
└── 设置与诊断
    ├── 连接器
    ├── 显示项目与顺序
    ├── 提醒
    ├── 隐私与权限
    └── 数据与诊断
```

---

## 11. 技术方案建议

### 11.1 客户端技术栈

- Swift 6 + SwiftUI；
- AppKit `NSStatusItem` 承载菜单栏；
- SwiftUI + AppKit 无边框 Panel 承载可交互悬浮便签；
- 后续 WidgetKit 组件通过 App Group 读取主应用已标准化的快照，不直接扫描第三方日志或读取凭证；
- SQLite 作为持久账本，避免仅依赖源日志生命周期；
- Keychain 保存需要持久化的授权材料；
- 文件事件监听 + 低频轮询兜底；
- Sparkle 或同等级签名更新方案；
- Developer ID 签名和 Apple 公证。

### 11.2 分发策略

首发建议采用官网 DMG + Homebrew Cask，而不是优先上 Mac App Store。

原因：读取 `~/.claude`、`~/.qoder`、`~/Library/Application Support/...` 等多个工具目录，在 App Sandbox 下会产生大量文件授权与兼容成本。未来可提供功能受限的 App Store 版，但不应拖累 MVP。

### 11.3 适配器协议

每个适配器至少实现：

```swift
protocol UsageAdapter {
    var id: String { get }
    var capabilities: Set<Capability> { get }

    func detect() async -> DetectionResult
    func requestAuthorization() async throws -> AuthorizationResult
    func collect(since cursor: CollectionCursor?) async throws -> CollectionBatch
    func diagnose() async -> DiagnosticReport
}
```

`CollectionBatch` 只返回标准化的 UsageEvent、QuotaSnapshot、CostEvent 和 ConnectorHealth，不返回原始对话内容。

### 11.4 采集方式分级

| 级别 | 方式 | 默认策略 |
|---|---|---|
| 1 | 被动读取本地结构化文件/数据库 | 自动检测，可默认开启 |
| 2 | 写入官方支持的 Hook / Notify | 明确展示改动并需用户确认 |
| 3 | 复用本地登录态访问官方/内部 usage API | 每个连接器单独授权并展示域名 |
| 4 | 用户提供 API Key | Keychain 保存，默认不导出 |
| 5 | 浏览器扩展同步网页账单 | v2 再评估 |

MVP 不使用网络中间人或 TLS 解密。

### 11.5 适配器稳定性

必须建设：

- 每个平台的脱敏 fixture；
- parser 契约测试；
- 数据库 schema 指纹；
- 版本与路径探测；
- 解析失败熔断，不写入错误的 0；
- last-good snapshot；
- 连接器健康状态；
- 远程下发“禁用有问题的适配器”和兼容性提示；
- parserVersion 与可重算历史。

远程配置只能下发声明式规则或禁用开关，不应无提示执行远程代码。

---

## 12. 隐私与安全要求

1. 默认完全本地存储。
2. 不读取、保存或上传提示词、回复正文、代码、工具参数和文件 diff。
3. 项目路径默认仅保存本机带盐哈希；UI 可在本机临时解析显示 basename。
4. SQLite 以只读方式访问；涉及 WAL 时先建立安全快照，不能锁住或修改原应用数据库。
5. 凭证只在内存中用于目标域名请求；需要保存时进入 Keychain。
6. 每个联网连接器必须显示访问域名、数据字段、刷新频率和关闭方式。
7. 内部/未公开 API 标记为“实验性”，并提供随时断开和清除凭证功能。
8. 导出诊断包前自动移除 token、cookie、完整路径、账号 ID、会话内容和请求正文。
9. 不做默认遥测；若未来引入崩溃报告，必须显式 opt-in。
10. 提供“删除全部本地数据”和“只重建索引”两种操作。

---

## 13. 非功能指标

| 指标 | MVP 目标 |
|---|---|
| 首次可见数据时间 | 已支持工具存在时小于 60 秒 |
| 本地增量刷新 | 新事件写入后 60 秒内出现 |
| 额度 API 刷新 | 默认 5–15 分钟，遵守 Retry-After |
| Token 准确度 | 对固定 fixture 与源数据误差不超过 1% |
| 重复计数 | 已知恢复/压缩/累计快照场景为 0 |
| 采集失败行为 | 保留 last-good 并标记过期，不显示伪 0 |
| 空闲 CPU | 平均低于 0.5% |
| 常驻内存 | 目标低于 120 MB |
| 崩溃率 | Crash-free sessions ≥ 99.8% |
| 隐私 | 诊断包和数据库中不含对话正文与凭证 |

---

## 14. 成功指标

### 北极星指标

**每周至少一次因产品提醒或建议而避免额度意外耗尽的活跃用户数。**

### 激活

- 首次启动 5 分钟内成功连接至少 2 个工具的比例 ≥ 70%；
- 自动检测成功率 ≥ 90%；
- 用户首次看到有效额度或使用量的中位时间 < 60 秒。

### 留存

- 第 4 周留存 ≥ 35%；
- 每周查看菜单栏 3 天以上的活跃用户比例 ≥ 50%。

### 信任

- “数字与平台明显不一致”的反馈率 < 2%；
- 连接器失败被正确标记为 stale/error 的比例 = 100%；
- 0 起凭证泄露和内容采集事件。

### 产品价值

- 至少 30% 周活用户启用额度提醒；
- 至少 20% 月活用户使用订阅复盘；
- 连接 4 个以上工具的用户占比持续增长。

---

## 15. 商业化建议

### 免费版

- 本地最多 5 个连接器；
- 菜单栏和当前额度；
- 7 日历史；
- 基础提醒；
- 本地导出。

### Pro 版

- 不限连接器；
- 1 年历史；
- 高级异常检测和耗尽预测；
- 订阅 ROI 分析；
- 自定义提醒规则；
- 多账号；
- 高级导出与自动报告。

### Team 版（v2）

- 仅同步聚合指标；
- 成员自行授权；
- 团队成本与采用率；
- 预算和异常提醒；
- 自托管或企业数据驻留选项。

不建议把基础准确性、隐私或连接器修复能力做成付费墙。

---

## 16. 里程碑与排期建议

假设配置：2 名 macOS/数据工程师 + 1 名产品设计/测试复合角色。

### M0：数据可行性验证（2 周）

- 获取 Qoder CLI / IDE / Work、WorkBuddy、CodeBuddy、千问办公的真实脱敏样本；
- 核对 Token、积分、额度字段；
- 建立数据模型、去重规则和 fixture；
- 对 TRAE CN 做单独 spike；
- 输出每个平台的可承诺能力矩阵。

退出条件：至少 5 个国内连接器能够稳定生成标准事件；无法获得真值的平台明确降级。

### M1：数据引擎与连接器（4 周）

- SQLite 持久账本；
- 增量游标、差分、去重和重算；
- 8 个 P0 连接器；
- 连接器健康状态和诊断；
- 价格表与成本类型分离。

### M2：菜单栏与桌面便签（3 周）

- 菜单栏卡片；
- 可选悬浮桌面便签；
- 轻量详情、连接器诊断和单张趋势图；
- 数字来源与可信度说明；
- 通知和耗尽预测。

### M3：封闭 Beta（2–3 周）

- 20–50 位多工具用户；
- 与各平台页面逐项对账；
- 修复版本兼容；
- 性能、签名、公证、自动更新；
- 隐私说明与诊断流程。

预计 11–12 周可交付可信的 MVP；如果首发同时承诺 TRAE CN、Comate 和豆包等不稳定来源，排期会明显失控。

---

## 17. 风险与应对

| 风险 | 影响 | 应对 |
|---|---|---|
| 平台升级导致日志/schema 改变 | 数据中断或错算 | schema 指纹、fixture、版本探测、last-good、快速禁用 |
| 内部 API 变化或违反平台条款 | 连接器失效、合规风险 | 显式授权、实验标签、优先官方/本地数据、法律审查 |
| 凭证读取引发信任问题 | 用户拒绝使用 | Keychain、域名白名单、可视化数据流、不开启即不联网 |
| 不同口径被错误相加 | 用户误判成本 | 强类型单位、Cost Type 分离、禁止默认跨单位汇总 |
| 本地日志被删除或重写 | 历史倒退 | 持久账本、事件指纹、源消息缓存 |
| 平台只给积分不给 Token | 无法横向比较 | 保留原单位，不制造 Token；仅在有公开倍率时给估算 |
| 竞品快速补齐国内平台 | 覆盖优势消失 | 聚焦可信度、适配器运维、预算决策和团队能力 |
| Mac App Sandbox 限制 | 大量授权、体验差 | 首发官网 DMG/Homebrew，后续评估受限商店版 |

---

## 18. MVP 验收标准

1. P0 八个连接器中至少六个达到可发布质量，其余必须明确显示 Beta/不可用，不能伪造数据。
2. 对同一 fixture 重复扫描十次，累计结果不变化。
3. 对累计快照、日志追加、会话恢复、日志删除、数据库 WAL 五类场景均有自动化测试。
4. 所有指标均能打开“数字解释”面板。
5. API/登录过期、文件不可读、schema 变化时，显示明确错误并保留 last-good。
6. 用户可以单独关闭任一连接器的文件读取、Hook 或联网能力。
7. 本地数据库、日志和诊断包不含凭证及对话正文。
8. 菜单栏能在 2 秒内打开，刷新时不阻塞 UI。
9. 应用完成 Developer ID 签名、公证和自动更新验证。
10. 至少 20 位 Beta 用户完成 7 天连续使用，并对主要平台完成一次人工对账。

---

## 19. 立项前必须回答的问题

1. “豆包工作”具体指哪个产品：豆包桌面端、TRAE/火山引擎相关产品，还是某个办公工作台？
2. 首发是纯个人工具，还是从第一天就考虑团队采购？
3. 是否接受读取本地登录凭证并调用平台内部 usage API？默认建议逐连接器 opt-in。
4. 是否计划开源？大量竞品为 MIT，开源有利于适配器审计和社区贡献，但会降低单纯靠覆盖面的壁垒。
5. 首发是否必须上 Mac App Store？如果必须，文件访问方案需要先单独验证。
6. 产品是只显示“使用量”，还是允许在额度不足时推荐或一键打开替代工具？后者更有决策价值。

---

## 20. 最终建议

建议立项，但需要调整最初命题：

> 做一款“中国平台覆盖最深、每个数字最可信、最能帮助用户避免额度耗尽”的 Mac 菜单栏与桌面便签小工具。

MVP 的胜负不取决于连接器总数，而取决于以下三点：

1. Qoder、WorkBuddy、CodeBuddy、千问办公四条国内主线是否真的算得准；
2. 用户是否能在 3 秒内看懂“还能用多少、何时耗尽、何时重置”；
3. 平台升级后，团队能否在 24–72 小时内定位并修复适配器。

如果这三点成立，再扩展 TRAE、Comate、CodeGeeX、豆包和团队管理，产品才有机会从“开源统计工具”升级为长期可持续的用量基础设施。

---

## 21. 主要调研来源

- [usageBar：国内工具与 Qoder 全家桶本地计量](https://github.com/ChanningYuan/usageBar)
- [TokenTracker：39 类工具、Hook/SQLite/JSONL 采集方式](https://github.com/xiufengsun/TokenTracker)
- [tokscale：多客户端统计与 TRAE 会话级同步](https://github.com/junhoyeo/tokscale)
- [OpenUsage.ai：原生 macOS ProviderRuntime 架构](https://github.com/robinebers/openusage)
- [OpenUsage.sh：跨平台用量、成本、burn rate 与 daemon](https://github.com/janekbaraniewski/openusage)
- [CC Switch：官方订阅、Token Plan、余额与自定义查询](https://github.com/farion1231/cc-switch/blob/main/docs/user-manual/en/2-providers/2.5-usage-query.md)
- [AI Usage：轻量原生菜单栏与 provider contracts](https://github.com/burakgon/ai-usage-menubar/blob/main/docs/provider-contracts.md)
- [Coding Usage Bar：GLM、DeepSeek、MiniMax、Kimi 与 pacing](https://github.com/hanzhangzzz/coding-usage-bar)
- [Qoder 官方：Usage & Quota](https://docs.qoder.com/cli/usage)
- [Qoder 官方：Team Usage API](https://docs.qoder.com/account/teams/openapi/usage)
- [CodeBuddy 官方：积分说明](https://www.codebuddy.cn/docs/ide/Account/credits)
