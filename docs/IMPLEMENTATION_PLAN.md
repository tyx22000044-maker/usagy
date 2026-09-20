# 实施计划

版本：v1.0-draft  
建议团队：2 名 macOS/数据工程师 + 1 名设计/测试复合角色  
目标周期：11–12 周

## 1. 开发策略

采用“纵向切片 + fixture 驱动”：先完成一条从源数据到菜单栏的完整路径，再并行扩展连接器。禁止先做八个半成品 parser，最后才验证 UI 和账本。

建议顺序：

1. Domain 与 fixture；
2. SQLite 账本；
3. Claude/Codex 两条成熟基线；
4. 菜单栏完整切片；
5. 桌面便签；
6. 国内连接器；
7. 通知、诊断和发布。

## 2. 工作流依赖

```text
P0 工程骨架
├── Domain 模型 ──► SQLite Ledger ──► Snapshot Service ──► UI
├── Adapter Protocol ──► Claude/Codex ──► 国内 Adapters
├── Design System ──► Menu Bar ──► Floating Card
└── Privacy/Logging ──► Diagnostics ──► Release
```

## 3. Phase 0：样本与风险验证（第 1–2 周）

### T0.1 创建工程骨架

- Swift 6；
- macOS 14+；
- App、Domain、Data、Adapters、Infrastructure、Features、DesignSystem targets；
- 单元测试与 UI 测试 targets；
- CI 执行 build/test/lint。

验收：空应用可构建运行，模块依赖方向正确，无反向 UI 依赖。

### T0.1A Swiss Ledger 基础层

- 从 1Cash / Footy 的真实实现复刻 `LedgerType`、`FamilyTypography`、`FamilyUI`、`AppSpacing`、`AppCornerRadius`、`AppStroke`、`AppMotion`；
- 打包 Archivo 五个字重及 OFL 许可；
- 创建 AI 用量表 `AccentColor`：`#C4321F` / `#E2503A`；
- 实现 SystemPanel、StatusBadge、Ledger buttons、StandardRow 和方形 AppSwitchStyle；
- 增加 DesignSystemTokenTests。

验收：token 测试通过；不存在 Capsule、内容阴影、10 pt 以上卡片圆角或系统 material；与 1Cash / Footy 参考截图并排确认家族一致性。

### T0.2 建立 fixture 工具链

- 定义 manifest/expected schema；
- 编写敏感字段扫描器；
- 建立 fixture loader；
- 准备 Claude 与 Codex 基线 fixture。

验收：测试可从 fixture 得到确定输出；敏感内容扫描失败时 CI 失败。

### T0.3 国内平台可行性 spike

分别确认 Qoder CLI/IDE/Work、WorkBuddy、CodeBuddy、千问办公：

- 实际路径；
- 版本；
- 数值字段；
- 去重键；
- 额度/积分来源；
- 是否需要 Hook 或网络；
- 隐私风险。

验收：每个平台形成一页 source contract；拿不到真值的能力明确降级。

## 4. Phase 1：数据内核（第 3–4 周）

### T1.1 Domain 模型

- 实现 DATA_MODEL 中核心类型；
- 强类型单位与 cost kind；
- evidence/freshness；
- validation。

### T1.2 SQLite Ledger

- migration 0001；
- usage/quota/cost/cursor/health repositories；
- 唯一索引与事务导入；
- daily aggregates 可重建。

### T1.3 Ingestion

- batch validation；
- dedup；
- cumulative delta；
- parserVersion；
- last-good；
- rollback。

### T1.4 Snapshot 与预测

- Popover snapshot query；
- 主指标选择；
- freshness；
- 基础 burn rate；
- 低置信预测不通知。

Phase 验收：给定 fixture，可重复导入并稳定生成菜单栏所需 snapshot。

## 5. Phase 2：第一个完整切片（第 5 周）

### T2.1 Claude Code Adapter

- detect、增量 JSONL、去重、fixture、health。

### T2.2 Codex Adapter

- rollout 增量、累计差分、fixture、health。

### T2.3 Menu Bar UI

- NSStatusItem；
- 360 pt Popover；
- Header、ProviderCard、Footer；
- 缓存先显示、后台刷新；
- loading/healthy/stale/error/empty。

### T2.4 基础设置

- 启用/隐藏/排序；
- 标题格式；
- 刷新；
- 开机启动。

Phase 验收：安装后可从两个真实工具读取并在菜单栏正确显示，关闭网络也可用。

## 6. Phase 3：桌面便签与通知（第 6 周）

### T3.1 Floating Panel

- Small/Medium；
- 拖动与位置恢复；
- 多屏/Spaces；
- 透明度与层级；
- 快捷键。

### T3.2 通知引擎

- 阈值；
- 去重 receipt；
- stale 抑制；
- 点击 deep link；
- 平台静音。

### T3.3 轻量详情与 Explain View

- 当前窗口；
- 7 日汇总；
- 单张趋势图；
- 来源/可信度/限制。

Phase 验收：菜单栏与便签共享同一 snapshot，切换显示不会启动重复采集器。

## 7. Phase 4：国内连接器（第 7–9 周）

按真实样本成熟度逐个完成：

### T4.1 Qoder CLI
### T4.2 Qoder IDE
### T4.3 Qoder Work
### T4.4 WorkBuddy
### T4.5 CodeBuddy
### T4.6 千问办公 / Qwen Work

每个任务的完成定义：

- detect；
- auth/permission；
- parser；
- cursor；
- health；
- 3 组 fixture；
- 契约测试；
- 真实对账；
- UI capability mapping；
- 隐私说明。

连接器之间可以并行，但 Ledger schema 与 Adapter Protocol 未稳定前不得开始大规模并行。

## 8. Phase 5：诊断、隐私与发布（第 10 周）

### T5.1 Connector Center

- 检测结果；
- 授权；
- 状态；
- 测试连接；
- 重新扫描；
- safe error。

### T5.2 Diagnostics

- 匿名诊断包；
- Redactor；
- schema/parser/version；
- 凭证与内容二次扫描。

### T5.3 数据管理

- 清索引；
- 断开连接器；
- 删除全部数据；
- 源应用数据绝不删除。

### T5.4 发布管线

- Developer ID；
- notarization；
- Sparkle/update feed；
- stable/beta channel；
- DMG；
- Homebrew Cask 草案。

## 9. Phase 6：封闭 Beta（第 11–12 周）

- 20–50 位多工具用户；
- 7 天使用；
- 每个平台至少 3 名真实用户；
- 与官方界面对账；
- 记录版本矩阵；
- 修复 P0/P1 问题；
- 性能与无障碍审查；
- 隐私说明和发布文案。

公开 Beta Gate：

- 至少 6 个连接器为 Supported；
- 无 P0/P1 数据准确性问题；
- crash-free internal sessions ≥99.8%；
- fixture/contract tests 全绿；
- 签名、公证、更新演练成功；
- 删除数据和撤销权限验证通过。

## 10. 任务拆分规范

每张开发票据必须包含：

- 用户/工程目标；
- 输入与输出；
- 依赖；
- 不做什么；
- 文件/模块范围；
- 状态与错误处理；
- 测试；
- 验收标准。

推荐单票 0.5–2 天。超过 3 天的任务必须继续拆分。

## 11. Agent 执行策略

单 Agent 顺序：按 Phase 逐项执行，每完成一个 Phase 更新文档与测试。

多 Agent 并行时建议：

- Agent A：Domain/Data/Ingestion；
- Agent B：MenuBar/FloatingCard/DesignSystem；
- Agent C：Adapters/Fixtures；
- 主 Agent：协议、集成、隐私和验收。

禁止多人同时修改核心 Domain 协议。先锁定协议，再分配连接器。

## 11.1 并行开发协作框架

详细协作规范参见 [AGENT_COLLABORATION.md](./AGENT_COLLABORATION.md)，以下为核心要点：

### Agent 角色定义

| Agent ID | 角色 | 职责范围 |
|----------|------|----------|
| Agent-Arch | 架构 Agent | Domain/, Data/, Infrastructure/ |
| Agent-UI | UI Agent | Features/, DesignSystem/ |
| Agent-Adapter | 适配器 Agent | Adapters/, Fixtures/ |
| Agent-Lead | 主控 Agent | 协议定义、集成、发布 |

### 分支管理

```text
分支命名：agent-{id}/{task-slug}
示例：agent-arch/domain-usage-event
```

### 同步机制

- 每日 09:00 从 develop pull/rebase
- 每日 18:00 创建 PR 到 develop
- Agent-Lead 每日审核并合并

### 冲突预防

- 接口定义由 Agent-Lead 锁定
- 代码所有权按目录划分
- 跨模块变更需双人审核

## 12. 首个开发 Prompt 模板

```text
请在 AIUsageMeter 工程中执行 IMPLEMENTATION_PLAN.md 的 T0.1。
开始前完整阅读 AGENTS.md、PRODUCT_SPEC.md、DECISIONS.md、ARCHITECTURE.md 和 TEST_PLAN.md。
只完成 T0.1，不实现连接器或业务 UI。创建 Swift 6 macOS 工程与模块骨架、测试 targets 和最小 CI。
完成后运行构建与测试，逐项对照 T0.1 验收标准，并列出新增文件、验证结果和仍待产品决定的事项。
```
