# Usagy Agent Instructions

这些规则适用于 `usagy/`（原 AIUsageMeter）下的所有实现工作。项目已更名为 **Usagy**。

## 1. 开始工作前

按顺序阅读：

1. `README.md`
2. `docs/PRODUCT_SPEC.md`
3. `docs/DECISIONS.md`
4. 与当前任务直接相关的设计、架构、数据或适配器文档
5. `docs/IMPLEMENTATION_PLAN.md` 中对应任务的依赖和验收标准
6. `docs/TEST_PLAN.md`
7. `docs/AGENT_COLLABORATION.md`（并行开发时必读）
8. `docs/DEVELOPMENT_GUIDE.md`（开发规范参考）
9. `docs/VERSION_CONTROL_STRATEGY.md`（版本控制规范）

如规格冲突，优先级为：用户最新指令 > `DECISIONS.md` > 专项规格 > `PRODUCT_SPEC.md` > 根目录调研 PRD。

## 2. 产品边界

- 产品首先是菜单栏与桌面便签小工具，不是重型 Dashboard。
- 任何主界面都应在 3 秒内回答：哪个工具最紧张、剩余多少、何时重置。
- MVP 是本地应用。本机采集、标准化、持久化属于“本地后端”；不得自行增加云服务器。
- 不做代理、不拦截模型流量、不实现聊天或供应商切换。
- 不读取或保存提示词、回复正文、代码正文、diff、工具参数和附件。
- 没有可靠真值时显示“不可用”或“估算”，不得制造精确数字。

## 3. 工程约束

- Swift 6 严格并发；SwiftUI 为主，AppKit 只承载菜单栏、悬浮 Panel 和系统能力缺口。
- UI 必须遵守 `docs/DESIGN_SPEC.md` 的 Swiss Ledger token；禁止自行引入玻璃材质、胶囊、阴影、大圆角或 Provider 品牌色卡片。
- 业务层不得依赖 SwiftUI、AppKit、SQLite 或具体连接器。
- 每个连接器必须实现统一协议，并通过同一套契约测试。
- 第三方数据库只读访问；不得修改、迁移或锁住源应用数据库。
- 凭证只放 Keychain 或瞬时内存；日志、fixture、错误信息不得出现原始凭证。
- 所有远程请求必须来自用户已开启的连接器，并限制到该连接器声明的域名。
- 数据采集失败时保留 last-good 值并标记 stale/error；不得落成 0。
- 所有累计型 telemetry 必须做差分；所有事件必须可幂等导入。

## 4. 修改纪律

- 一个任务只解决一个明确目标，不顺手扩张范围。
- 新增用户可见能力前，先补产品状态、空态、错误态和权限态。
- 新增连接器前，先提交脱敏 fixture、来源说明和预期输出，再写 parser。
- 数据模型变化必须同步更新 `DATA_MODEL.md`、迁移测试和 fixture 版本。
- 关键架构决策必须追加到 `DECISIONS.md`，不得只存在于代码注释或聊天中。
- 不覆盖用户已有配置，不自动写 Hook；任何 Hook 安装必须预览、确认、可恢复。

## 5. 完成定义

任务完成必须同时满足：

- 功能符合对应规格和验收标准；
- 单元/契约/UI 测试按风险补齐；
- `swift test` 和相关 Xcode 测试通过；
- 无凭证、用户内容和绝对路径泄漏；
- VoiceOver、键盘操作、浅色/深色和数据过期状态已检查；
- 文档与实现一致；
- 未完成或依赖真实样本的内容以明确 TODO 记录，不得假实现。

## 6. 禁止事项

- 禁止把估算成本命名为"实际花费"。
- 禁止把 Token、积分、请求和金额直接相加。
- 禁止在 UI 中用空值或解析失败冒充 0。
- 禁止远程下发并执行任意代码。
- 禁止静默读取浏览器 Cookie 或系统凭证。
- 禁止为了"支持平台"而使用未经验证的字段猜测。

## 7. 并行开发协作

详细规范参见 `docs/AGENT_COLLABORATION.md`，核心规则：

### 7.1 Agent 身份与职责

- 每个 Agent 必须明确自己的角色（Arch/UI/Adapter/Lead）
- 只修改职责范围内的代码目录
- 跨模块接口变更需 Agent-Lead 审核

### 7.2 分支与提交

- 分支命名：`agent-{id}/{task-slug}`
- 提交信息格式：`<type>(<scope>): <subject>` + Footer（Agent/Task/Refs）
- 每日至少提交一次进展

### 7.3 同步与冲突预防

- 每日 09:00 从 develop pull/rebase
- 接口定义由 Agent-Lead 锁定，变更需 RFC
- 代码所有权按目录划分，遵循 CODEOWNERS

### 7.4 评审与合并

- 所有代码通过 PR 合并到 develop
- 功能分支使用 Squash Merge
- 至少一人评审通过方可合并

### 7.5 问题升级

- Level 1：Agent 自行解决
- Level 2：协作 Agent 协商
- Level 3：Agent-Lead 仲裁
- Level 4：用户决策
