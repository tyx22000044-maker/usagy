# 数据模型

版本：v1.0-draft

## 1. 建模原则

- 客户端、账号、套餐、模型供应商和模型分离；
- 事件与快照分离；
- 原生单位优先，不跨单位强行汇总；
- 来源、可信度、时间和 parser 版本属于数据本身；
- 导入必须幂等；
- 未知是 `nil/unavailable`，不是 0。

## 2. 核心枚举

```swift
enum MetricUnit: String, Codable, Sendable {
    case token, credit, request, percent
    case usd, cny
}

enum TokenKind: String, Codable, Sendable {
    case input, output, cacheRead, cacheWrite, reasoning
}

enum CostKind: String, Codable, Sendable {
    case billed
    case subscriptionAllocated
    case apiEquivalentEstimate
    case vendorReportedEstimate
}

enum EvidenceLevel: Int, Codable, Sendable {
    case officialBilling = 5
    case officialQuota = 4
    case localStructured = 3
    case hookStructured = 2
    case estimated = 1
}

enum FreshnessState: String, Codable, Sendable {
    case fresh, stale, expired, unknown
}
```

## 3. 领域实体

### ClientTool

表示实际客户端产品，如 Qoder IDE 或 WorkBuddy。

字段：

- `id`：稳定内部 ID，例如 `qoder.ide.cn`；
- `familyID`：产品家族，例如 `qoder`；
- `displayName`；
- `region`：global/cn/unknown；
- `installationVariant`；
- `detectedVersion`；
- `iconAssetName`。

### Account

- `id`：本机生成 UUID；
- `connectorID`；
- `externalAccountHash`：带盐哈希，可空；
- `displayLabel`：脱敏昵称，可空；
- `region`；
- `planID`；
- `firstSeenAt` / `lastSeenAt`。

同一平台不同账号必须分开；无法可靠识别账号时使用 connector-scoped anonymous account，并标记低置信。

### ModelIdentity

- `canonicalID`；
- `reportedID`；
- `providerID`；
- `displayName`；
- `pricingVersion`；
- `isUnknown`。

### UsageEvent

- `id`：UUID；
- `dedupKey`：稳定唯一键；
- `connectorID`；
- `clientToolID`；
- `accountID`；
- `sessionHash` / `requestHash` / `projectHash`，均可空；
- `modelCanonicalID` / `modelReportedID`；
- `occurredAt`；
- `inputTokens`；
- `outputTokens`；
- `cacheReadTokens`；
- `cacheWriteTokens`；
- `reasoningTokens`；
- `requestCount`；
- `creditUsed`；
- `sourceKind`；
- `evidenceLevel`；
- `parserVersion`；
- `importedAt`。

Token 字段均为 nullable 非负整数。源数据未提供时为 null，不得填 0。

### QuotaSnapshot

- `id`；
- `dedupKey`；
- `connectorID` / `accountID`；
- `quotaKey`：平台内稳定键；
- `displayName`；
- `unit`；
- `used` / `remaining` / `total`，均可空；
- `percentageSemantic`：used/remaining；
- `windowStart` / `windowEnd` / `resetAt`；
- `capturedAt`；
- `evidenceLevel`；
- `sourceKind`；
- `isEstimated`；
- `parserVersion`。

约束：used、remaining、total 至少一个非空；如果可以推导，只在展示层推导，不反写为平台原始值。

### CostEvent

- `id`；
- `usageEventID`，可空；
- `connectorID` / `accountID`；
- `kind`；
- `currency`；
- `amountMinor`：最小货币单位或 Decimal 字符串；
- `pricingSource`；
- `pricingVersion`；
- `occurredAt`；
- `isEstimated`。

不得使用二进制浮点储存货币。

### ConnectorCursor

- `connectorID`；
- `sourceID`；
- `cursorType`；
- `cursorData`：版本化 JSON；
- `sourceFingerprint`；
- `updatedAt`。

### ConnectorHealth

- `connectorID`；
- `state`；
- `lastAttemptAt`；
- `lastSuccessAt`；
- `consecutiveFailureCount`；
- `errorCategory`；
- `safeMessage`；
- `detectedVersion`；
- `schemaFingerprint`；
- `parserVersion`；
- `cooldownUntil`。

## 4. SQLite 表

建议表：

```text
client_tools
accounts
models
usage_events
quota_snapshots
cost_events
connector_cursors
connector_health
daily_aggregates
notification_receipts
settings
schema_migrations
```

关键索引：

- `usage_events(dedup_key)` UNIQUE；
- `usage_events(connector_id, occurred_at)`；
- `usage_events(account_id, occurred_at)`；
- `usage_events(model_canonical_id, occurred_at)`；
- `quota_snapshots(dedup_key)` UNIQUE；
- `quota_snapshots(account_id, quota_key, captured_at DESC)`；
- `daily_aggregates(day, connector_id, account_id, model_canonical_id)` UNIQUE；
- `notification_receipts(account_id, quota_key, window_end, threshold)` UNIQUE。

## 5. 去重键

优先级：

1. 平台稳定 request ID；
2. message ID + usage vector identity；
3. session ID + monotonic sequence；
4. source file identity + byte offset；
5. 标准化字段哈希。

标准化哈希至少包含 connector、account、model、timestamp bucket、token vector 和 source identity。不得包含提示词或回复正文。

## 6. 累计快照差分

对于 Codex 等累计 telemetry：

```text
delta = current_total - previous_total
```

规则：

- 差分按同一 source/account/session/metric 计算；
- current < previous 时视为新 epoch/reset，不产生负数；
- 首个累计点默认只建立 baseline，除非来源明确说明它代表完整可计费事件；
- parserVersion 变化不得重复导入旧 delta；
- fixture 必须覆盖乱序、重复、reset、缺失中间点。

## 7. 日聚合

`daily_aggregates` 是可重建缓存，不是事实来源。

维度：

- calendar day + timezone；
- connector；
- account；
- client tool；
- model；
- evidence level。

聚合不得跨单位。Token 允许分别求和，积分与货币保持独立列。

## 8. Freshness

每类源定义 freshness policy：

- 本地实时日志：2 分钟内 fresh，30 分钟后 stale；
- 网络 quota：15 分钟内 fresh，60 分钟后 stale；
- 手工同步：显示 capturedAt，不自动判失败；
- resetAt 已过去但没有新快照：expired。

UI 只对 fresh quota 做耗尽预测。

## 9. Forecast

耗尽预测输入：

- fresh quota remaining；
- 最近 30–120 分钟同单位使用速度；
- 至少 3 个有效数据点；
- 排除 reset、导入历史和异常大批量回补。

输出：

- `estimatedExhaustionAt`；
- `confidence`：low/medium/high；
- `sampleWindow`；
- `reasonUnavailable`。

低置信预测只在详情显示，不触发通知。

## 10. 隐私字段策略

- External ID：SHA-256(machineSalt + connector + rawID)；
- 项目：默认保存哈希与 basename，完整路径不入库；
- Session/Request：只存带盐哈希；
- 账号昵称：用户选择显示时保存脱敏文本，否则为空；
- 原始 payload：不持久化；测试 fixture 必须人工脱敏。

## 11. Migration 规则

- 每个 migration 单向、编号、可重复验证；
- 测试从空库和上一公开版本升级；
- 删除列/表前至少跨一个版本保留；
- 领域字段语义变化必须新建字段或显式重算，不能静默复用；
- migration 失败时原库保持可恢复，并提示用户导出诊断。

