# 连接器与适配器规范

版本：v1.0-draft

## 1. 目标

适配器负责把某个工具的本地日志、数据库、Hook 或额度接口转换为统一领域事件。它不负责 UI、长期聚合、通知或跨平台业务规则。

## 2. 能力模型

```swift
enum AdapterCapability: String, Sendable {
    case tokenUsage
    case requestUsage
    case creditUsage
    case quota
    case resetTime
    case billedCost
    case estimatedCost
    case modelBreakdown
    case projectAttribution
    case sessionAttribution
    case multiAccount
}
```

连接器声明能力不等于当前账号一定有数据。采集结果需要逐指标返回 available/unavailable/error。

## 3. 协议

```swift
protocol UsageAdapter: Sendable {
    var descriptor: AdapterDescriptor { get }

    func detect(context: DetectionContext) async -> DetectionResult
    func authorizationStatus(context: AdapterContext) async -> AuthorizationStatus
    func collect(context: AdapterContext,
                 cursor: CollectionCursor?) async throws -> CollectionBatch
    func diagnose(context: AdapterContext) async -> AdapterDiagnostic
}
```

`AdapterDescriptor`：

- id、displayName、familyID、region；
- capabilities；
- sourceKinds；
- candidatePaths；
- allowedHosts；
- parserVersion；
- supportedVersionRange；
- defaultRefreshPolicy；
- privacyDisclosure。

`CollectionBatch`：

- usageEvents；
- quotaSnapshots；
- costEvents；
- nextCursor；
- warnings；
- sourceFingerprint；
- collectedAt；
- completeness。

## 4. 来源类型

### Passive File

- JSONL、JSON、结构化日志；
- 只读；
- 记录 inode/size/mtime/offset；
- 文件变小或 identity 改变时开启新 epoch；
- 单行损坏不应终止整个文件，但需要统计并告警。

### Passive SQLite

- 使用只读连接或安全快照；
- 首先验证 schema fingerprint；
- 查询只选择数值 telemetry 必需列；
- 不读取对话正文列，即使数据库中存在。

### Hook / Notify

- 只有源工具公开或稳定支持 Hook 时使用；
- 安装前展示变更 diff；
- 写入必须原子、备份、幂等；
- 卸载只删除带本应用 ownership marker 的内容；
- 已有同名用户 Hook 不覆盖。

### Account API

- 明确授权后启用；
- host allowlist；
- 凭证不复制到日志/数据库；
- 401/403 最多刷新一次；
- 429 遵守服务端退避；
- 返回的 vendor cost 与本地估算分开。

## 5. 检测规则

检测不得产生写操作或网络请求。结果包含：

- installation found；
- version；
- source candidates；
- auth material exists（仅布尔值）；
- required permission；
- confidence；
- safe explanation。

仅目录存在不足以判定“可用”；至少还需存在预期文件/schema/可执行版本之一。

## 6. 错误分类

```swift
enum AdapterErrorCategory: String {
    case permissionDenied
    case authenticationExpired
    case rateLimited
    case networkUnavailable
    case unsupportedVersion
    case schemaMismatch
    case malformedSource
    case sourceBusy
    case credentialUnavailable
    case serviceUnavailable
    case internalInvariant
}
```

适配器错误必须包含安全用户文案、可重试性、建议动作和底层调试码。底层错误字符串在展示前必须脱敏。

## 7. 连接器发布门槛

标记“Supported”必须具备：

- 至少 3 份不同版本/账户场景的脱敏 fixture；
- 正常、重复、损坏、空值、版本变化测试；
- 数据路径与字段来源说明；
- 可信等级；
- 失败降级策略；
- 隐私审查；
- 一名真实用户与官方界面对账。

不足时只能标记 Beta/Experimental/Research。

## 8. P0 适配器契约

### Claude Code

来源：`~/.claude/projects/**/*.jsonl`；可选官方额度接口。

要求：

- 按 message/request identity 去重；
- 分离 input/output/cache read/cache write；
- 不读取消息正文；
- 额度凭证访问单独授权；
- 本地 Token 与官方额度是两个独立数据源。

### Codex

来源：rollout JSONL / 本地状态；可选官方额度接口。

要求：

- 优先 per-turn usage；
- 只有 cumulative 时做差分；
- cached input 是 input 子集时映射为互斥口径；
- 处理 session reset 和乱序；
- API Key 登录不能假设可读取订阅额度。

### Qoder CLI

候选来源：`~/.qoder/projects/**/*.jsonl`。

要求：

- 识别四类 Token；
- 暴露 Token 开关未开启时显示明确引导；
- 只对开启后的新请求承诺可见；
- CN 与国际版不合并。

### Qoder IDE

候选来源：`~/Library/Application Support/Qoder/SharedClientCache/**/local.db`，CN 使用独立目录。

要求：

- 只读取 assistant usage/token_info 数值字段；
- schema fingerprint；
- WAL 安全读取；
- 不读取 chat content；
- 同一消息只计一次。

### Qoder Work

候选来源：应用日志和本地 mirror。

要求：

- 真实样本验证后确定 parser；
- 日志轮转与重复 mirror 去重；
- 无 cache 字段时保持 null；
- 与 Qoder 账户额度合并展示，但客户端使用事件分开。

### WorkBuddy

候选来源：`~/.workbuddy/projects/**/*.jsonl`、本地 usage/credit 存储。

要求：

- Token 与 credit 分开；
- Claude 风格记录按消息 identity 去重；
- 若账户额度来源不稳定，只显示已观测用量；
- 不将平台 credit 估算成 Token。

### CodeBuddy

候选来源：SessionEnd Hook、本地结构化记录、经授权的账户资源。

要求：

- Hook 具备 ownership marker 和恢复；
- 精确余额不可用时不显示剩余 0；
- credit usage 与 Token 字段分开；
- 平台内部 API 必须 Experimental + opt-in。

### 千问办公 / Qwen Work

候选来源：本地 segment；可选账单/积分接口。

要求：

- 请求级去重；
- input/output/cache read 可用性逐字段表达；
- 本地 Token 与账单积分不相加；
- 账单同步单独联网授权。

## 9. Fixture 格式

```text
Fixtures/<adapter>/<case>/
├── manifest.json
├── source/...
└── expected.json
```

`manifest.json` 至少包含：

- adapterID；
- sourceVersion；
- capturedPlatform；
- sanitizationVersion；
- scenario；
- expectedWarnings；
- forbiddenContentScanPassed。

fixture 不得含真实用户名、路径、账号、凭证、提示词或回复内容。正文列可替换为固定占位符或删除。

## 10. 契约测试

所有适配器复用以下测试：

- detect 不写文件、不联网；
- collect 可重复运行且结果幂等；
- cursor 后只返回增量；
- malformed record 不泄漏原文；
- unknown schema 失败关闭；
- null 保持 null；
- 负 Token/金额被拒绝；
- 凭证不进入日志；
- 取消任务能及时退出；
- large fixture 不阻塞主线程。

