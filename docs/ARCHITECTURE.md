# 技术架构

版本：v1.0-draft

## 1. 架构目标

- 原生、轻量、低常驻资源；
- UI 打开即显示缓存，不等待采集；
- 连接器彼此隔离，一个失败不影响其他平台；
- 所有导入幂等，可从 fixture 重放；
- 业务模型与具体日志/API 解耦；
- 不引入云后端；
- 为未来 WidgetKit、CLI 和团队同步保留清晰边界。

## 2. 系统上下文

```text
第三方 AI 工具
├── JSONL / SQLite / 应用日志
├── Hook / Notify 事件
└── Usage / Quota API（显式授权）
          │
          ▼
Adapter Runtime
├── Detect
├── Authorize
├── Collect
├── Normalize
└── Diagnose
          │
          ▼
Usage Ingestion Service
├── Validate
├── Deduplicate
├── Cumulative Delta
├── Cost Classification
└── Persist
          │
          ▼
Local Ledger (SQLite)
          │
          ▼
Snapshot / Forecast Services
          │
     ┌────┴───────────┐
     ▼                ▼
Menu Bar UI     Floating Card UI
```

## 3. 模块划分

### App

- 应用生命周期；
- `NSStatusItem` 和 Popover/Panel 管理；
- 依赖注入；
- URL/通知入口；
- accessory activation policy。

### Domain

纯 Swift 类型和规则，不依赖 UI、数据库或网络：

- UsageEvent、QuotaSnapshot、CostEvent；
- MetricValue、Unit、Confidence；
- ConnectorState；
- 去重键和累计差分规则；
- Pacing / exhaustion forecast；
- 展示优先级。

### Data

- SQLite schema 和 migration；
- repository；
- 查询与聚合；
- last-good snapshot；
- retention 和 purge。

### Adapters

- 统一协议；
- 每个平台独立目录；
- parser、credential locator、API client、fixture；
- 不包含 SwiftUI。

### Infrastructure

- FileWatcher；
- NetworkClient；
- KeychainStore；
- Logger/Redactor；
- NotificationScheduler；
- LaunchAtLogin；
- UpdateService；
- Clock 和 UUID 注入。

### Features

- MenuBar；
- FloatingCard；
- Detail；
- Connectors；
- Settings；
- Onboarding。

### DesignSystem

- 颜色、间距、字体；
- MetricRing、UsageBar、StatusBadge、ProviderIcon；
- Card、EmptyState、ErrorBanner。

## 4. 推荐代码结构

```text
Sources/
├── App/
│   ├── AIUsageMeterApp.swift
│   ├── AppDelegate.swift
│   ├── AppEnvironment.swift
│   └── WindowCoordinator.swift
├── Domain/
│   ├── Models/
│   ├── Services/
│   ├── Protocols/
│   └── Policies/
├── Data/
│   ├── Database/
│   ├── Repositories/
│   ├── Migrations/
│   └── Queries/
├── Adapters/
│   ├── Shared/
│   ├── ClaudeCode/
│   ├── Codex/
│   ├── QoderCLI/
│   ├── QoderIDE/
│   ├── QoderWork/
│   ├── WorkBuddy/
│   ├── CodeBuddy/
│   └── QwenWork/
├── Infrastructure/
├── Features/
└── DesignSystem/
```

## 5. 并发模型

采用 Swift 6 严格并发：

- `ConnectorRuntime` 为 actor，负责连接器状态和调度；
- 每个连接器采集任务为独立 child task；
- `LedgerWriter` 为 actor，串行化数据库写入；
- 数据库只读查询通过独立 repository 执行；
- UI ViewModel 标记 `@MainActor`；
- parser 尽量为纯函数 + `Sendable` 输入输出；
- 所有长扫描分批 yield，禁止阻塞 MainActor；
- App 退出时取消采集，不强制等待长网络请求。

建议接口：

```swift
actor ConnectorRuntime {
    func startEnabledConnectors()
    func refresh(_ connectorID: ConnectorID, reason: RefreshReason) async
    func refreshAll(reason: RefreshReason) async
    func stateStream() -> AsyncStream<[ConnectorStatus]>
}
```

## 6. 数据流

### 启动

1. 打开数据库并执行 migration；
2. 读取 last-good UI snapshot；
3. 创建菜单栏并立即显示缓存；
4. 启动已启用连接器；
5. 后台执行 detect/collect；
6. 写入账本并发布新 snapshot；
7. UI 观察 snapshot stream 更新。

### 本地文件变更

1. FileWatcher 收到事件；
2. 按连接器 1.5 秒 debounce；
3. 连接器从持久 cursor 增量读取；
4. parser 输出标准事件；
5. ingestion 校验、去重、差分；
6. 单事务写入；
7. 重算受影响日期/窗口；
8. 发布 UI snapshot。

### 网络额度刷新

1. scheduler 检查 connector cooldown；
2. 从 Keychain 或源应用凭证位置读取授权；
3. 请求 allowlist 域名；
4. 解析为 QuotaSnapshot；
5. 更新 last-good；
6. 401/403 至多刷新凭证并重试一次；
7. 429 遵守 `Retry-After`；
8. 临时失败保留旧值并标记 stale。

## 7. 本地服务边界

MVP 不运行 HTTP Server。UI 与采集层处于同一进程，通过协议与 actor 通信。

未来如增加 CLI/WidgetKit：

- WidgetKit 读取 App Group 中的脱敏展示快照；
- CLI 可通过只读打开同一 SQLite，或后续增加 XPC；
- 不为方便而在 localhost 暴露无认证 API。

## 8. 数据库选型

使用 SQLite。可采用 GRDB，但 Domain 不得暴露 GRDB 类型。

要求：

- WAL 模式用于本应用自己的数据库；
- migration 版本化且可测试；
- 所有写入使用事务；
- 事件表以稳定 dedup key 建唯一索引；
- 原始第三方 payload 默认不持久化；
- 诊断所需字段必须先脱敏；
- 数据库损坏时先隔离副本，再创建新库，不覆盖源应用数据。

## 9. 源数据库读取

- 优先 SQLite URI `mode=ro`；
- 对活跃 WAL 数据库，使用 SQLite backup API 或复制 db/wal/shm 到本应用临时目录后读取；
- 临时文件使用随机目录并及时删除；
- 不执行 `VACUUM`、migration、pragma 写操作；
- 设置合理 busy timeout，不与源应用争锁；
- schema 指纹不匹配时停止解析并报告 unsupportedVersion。

## 10. 日志与隐私

统一结构化日志，但只包含：

- connectorID；
- operation；
- duration；
- count；
- parserVersion；
- errorCategory；
- 脱敏路径标识。

禁止记录：

- token/cookie/API key；
- 提示词、回复、代码；
- 完整 home 路径；
- 原始账号 ID；
- 完整请求/响应 body。

Debug 构建也遵守此规则。

## 11. 性能预算

- 启动到菜单栏可交互：<1 秒；
- Popover 打开：缓存路径 <100 ms；
- 空闲 CPU 平均 <0.5%；
- 常驻内存目标 <120 MB；
- 单次增量扫描不读取未变化的大文件；
- 大型首次导入每 25–100 个文件 yield；
- 网络连接器同时并发最多 3 个；
- 每连接器独立 exponential backoff。

## 12. 失败隔离

- 每个连接器独立状态机；
- parser 崩溃不得导致应用崩溃；Swift 内应转换为 typed error；
- 数据校验失败的 batch 整批回滚；
- 单条坏记录可隔离并计数，但必须达到阈值后将连接器标记 degraded；
- schema 不兼容时停止新增，不猜测字段；
- 通知和预测只使用 fresh/valid 数据。

## 13. 安全边界

- 所有凭证访问通过 `CredentialStore`；
- 网络请求通过注入的 `ConnectorHTTPClient`，强制 host allowlist；
- 重定向到未声明域名时失败；
- TLS 使用系统实现；
- 不接受远程可执行脚本；
- Hook 安装前展示 diff，并保留可恢复备份；
- 导出诊断运行统一 Redactor 并进行二次敏感字段扫描。

## 15. 并行开发架构

### 15.1 模块隔离原则

为支持多 Agent 并行开发，架构设计遵循以下隔离原则：

1. **目录级隔离**：每个模块目录由单一 Agent 独占管理
2. **接口锁定**：跨模块接口通过协议定义并锁定版本
3. **依赖方向**：严格单向依赖，避免循环引用
4. **测试隔离**：每个模块可独立编译和测试

### 15.2 接口版本管理

所有跨模块接口必须遵循版本化管理：

```swift
// MARK: - Interface Versioning
// 版本号格式: Major.Minor
// Major: 不兼容变更
// Minor: 向后兼容变更
// 状态: LOCKED / DRAFT / DEPRECATED

// 示例：
// Domain/Protocols/UsageAdapter.swift
// 版本: v1.0
// 状态: LOCKED
// 变更需 RFC-XXX 批准
```

### 15.3 并行开发数据流

```text
并行开发时的数据流隔离：

Agent-Arch (Domain/Data)
  ↓ 定义接口
Agent-Lead (审核锁定)
  ↓ 提供接口
Agent-UI (Features) ←→ Agent-Adapter (Adapters)
  ↓ 实现接口        ↓ 实现接口
Agent-Lead (集成测试)
```

### 15.4 模块依赖矩阵

```text
依赖方向（箭头表示依赖）：

App → Domain, Data, Infrastructure, Features
Features → Domain, DesignSystem, Infrastructure
Adapters → Domain, Infrastructure
Data → Domain
Infrastructure → Domain（仅协议）
Domain → 无依赖（纯类型）
```

### 15.5 并行编译策略

为支持并行编译，采用以下策略：

1. **模块化编译**：每个模块独立编译为 framework
2. **增量编译**：仅重编译变更模块
3. **缓存共享**：编译缓存可在 Agent 间共享
4. **依赖图优化**：最小化模块间依赖

### 15.6 测试隔离策略

```text
测试隔离要求：

单元测试：
  - 每个模块独立测试 target
  - 不依赖其他模块实现
  - 使用 mock/stub 隔离依赖

集成测试：
  - Agent-Lead 负责维护
  - 验证模块间接口兼容性
  - 使用真实接口但 mock 外部依赖

契约测试：
  - 每个适配器独立契约测试
  - 验证接口实现正确性
  - 使用 fixture 驱动
```

