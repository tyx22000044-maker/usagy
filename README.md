# Usagy

`Usagy`（原 AIUsageMeter）是一款常驻 macOS 菜单栏、并可选择显示桌面悬浮便签的轻量 AI 用量工具。视觉完整继承 1Cash / Footy 的 Swiss Ledger 体系。

本目录是一套可直接交给开发 Agent 执行的产品与工程规格。任何实现都应先阅读 [AGENTS.md](./AGENTS.md)，再按以下顺序阅读：

1. [产品规格](./docs/PRODUCT_SPEC.md)
2. [产品与技术决策](./docs/DECISIONS.md)
3. [设计规格](./docs/DESIGN_SPEC.md)
4. [技术架构](./docs/ARCHITECTURE.md)
5. [数据模型](./docs/DATA_MODEL.md)
6. [适配器规范](./docs/ADAPTER_SPEC.md)
7. [实施计划](./docs/IMPLEMENTATION_PLAN.md)
8. [测试计划](./docs/TEST_PLAN.md)
9. [Agent 协作框架](./docs/AGENT_COLLABORATION.md)
10. [开发指南](./docs/DEVELOPMENT_GUIDE.md)
11. [版本控制策略](./docs/VERSION_CONTROL_STRATEGY.md)

原始市场调研与完整 PRD 位于 [AI-Usage-Monitor-PRD.md](../AI-Usage-Monitor-PRD.md)。若两份文档发生冲突，以本目录内更具体、版本更新的规格为准。

## MVP 交付物

- 无 Dock 图标的 macOS 菜单栏应用；
- 点击菜单栏后出现轻量 Popover；
- 可选悬浮桌面便签；
- 本地 SQLite 持久账本；
- 连接器自动检测、健康状态、权限说明和诊断；
- 首批可发布连接器：Claude Code、Codex，以及经样本验证后的 Qoder、WorkBuddy、CodeBuddy、千问办公连接器；
- 额度阈值、预计耗尽、重置和连接失效通知；
- Developer ID 签名、公证与更新机制。

## 明确不做

- 云端账号系统；
- Web 后端或远程数据库；
- 模型请求代理/MITM；
- 供应商切换与 API Key 管理器；
- 聊天、项目管理或会话管理；
- 重型数据分析 Dashboard；
- 读取或保存提示词、回复正文、代码和工具参数。

## 建议工程结构

```text
usagy/
├── Package.swift
├── Usagy.xcodeproj
├── Sources/
│   ├── App/
│   ├── Features/
│   │   ├── MenuBar/
│   │   ├── FloatingCard/
│   │   ├── Detail/
│   │   ├── Connectors/
│   │   └── Settings/
│   ├── Domain/
│   ├── Data/
│   ├── Adapters/
│   ├── Infrastructure/
│   └── DesignSystem/
├── Tests/
│   ├── DomainTests/
│   ├── DataTests/
│   ├── AdapterContractTests/
│   └── UITests/
├── Fixtures/
└── docs/
```

工程尚未创建时，实施 Agent 应从 `IMPLEMENTATION_PLAN.md` 的 Phase 0 开始，不得跳过 fixture 和数据契约直接堆叠 UI。