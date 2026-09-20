# 测试与质量计划

版本：v1.0-draft

## 1. 质量目标

优先级：数据正确性 > 隐私安全 > 稳定性 > 性能 > 视觉细节。

任何“错误但看起来精确”的数字都是发布阻断问题。

## 2. 测试层级

### Unit

- 数值校验；
- 单位与成本类型；
- 去重键；
- cumulative delta；
- freshness；
- forecast；
- 主指标排序；
- 通知去重。

### Adapter Contract

- detect 只读；
- fixture parse；
- cursor 增量；
- 重复导入；
- schema mismatch；
- malformed/null/negative；
- 取消、超时、429、401；
- 日志脱敏。

### Repository / Migration

- 空库建表；
- 上一版本升级；
- 事务回滚；
- 唯一索引；
- 可重建 aggregate；
- corruption recovery；
- 并发读写。

### Integration

- source fixture → adapter → ingestion → ledger → UI snapshot；
- 文件追加、轮转、截断、删除；
- SQLite WAL；
- 网络 last-good/stale；
- Hook 安装/卸载/用户已有配置保护。

### UI

- 首次启动；
- Popover 打开/关闭；
- 6+ 平台滚动；
- 显示/隐藏/排序；
- Small/Medium 便签；
- 多屏恢复；
- 通知 deep link；
- 设置与清除数据。

### Snapshot / Visual

组合：

- 浅色/深色；
- 英文/简中；
- 正常/注意/危险/stale/error/empty；
- 1、3、6、12 平台；
- 0.5%、9%、100%、长积分值；
- Increase Contrast/Reduce Transparency。

Swiss Ledger 额外检查：

- 与 1Cash Ledger / Footy SystemPanel 并排对比；
- 2 pt 方角与直角进度；
- Archivo 字重和 CJK fallback；
- 无 Capsule、内容阴影、玻璃 material 和 Provider 品牌色背景；
- 数值等宽对齐；
- 状态标签为描边矩形且不填色。

### Accessibility

- VoiceOver label/value/hint；
- 键盘顺序；
- Reduce Motion；
- 不依赖颜色；
- 最小点击区域。

### Performance

- 10k、100k、1m 事件导入；
- 10 GB 等效日志目录的增量扫描模拟；
- Popover 缓存打开 <100 ms；
- 空闲 CPU <0.5%；
- memory <120 MB 目标；
- 文件风暴 debounce；
- 8 连接器同时启动。

### Security / Privacy

- fixture 敏感字符串扫描；
- 日志凭证扫描；
- 重定向 host allowlist；
- Keychain 读写；
- 诊断包 redaction；
- 完整路径/账号 ID 哈希；
- 禁止提示词/回复字段进入数据库；
- 删除数据后残留检查。

## 3. 必测数据场景

1. 同一事件重复出现；
2. 同一响应分 thinking/text 两行；
3. 累计快照连续增长；
4. 累计值 reset；
5. 乱序时间戳；
6. 文件截断与原地重写；
7. 日志轮转；
8. 会话恢复/压缩；
9. cache 字段缺失；
10. 只有积分没有 Token；
11. 只有额度没有本地事件；
12. used/remaining 百分比语义相反；
13. resetAt 已过去但刷新失败；
14. API 401/403/429/5xx；
15. schema 增删字段；
16. 超大整数与货币精度；
17. 多账号混合；
18. 时区和夏令时切换；
19. 机器睡眠后恢复；
20. 源工具未安装或已卸载。

## 4. 发布阻断等级

### P0

- 凭证或内容泄漏；
- 修改/损坏源工具数据；
- 大范围重复计费或负数；
- 应用无法启动；
- 更新包签名失败。

### P1

- 主要连接器误差 >1%；
- 解析失败显示为 0；
- stale 数据触发错误通知；
- Popover 频繁卡死/崩溃；
- 删除数据或撤销授权无效。

### P2

- 单个平台边缘版本不兼容但已正确报错；
- 次要视觉问题；
- 非核心设置丢失。

公开 Beta 不允许存在已知 P0/P1。

## 5. Fixture 审核

每份 fixture 合入前：

- 两人审查或自动化 + 人工复核；
- 确认无凭证、真实内容和完整路径；
- manifest 记录来源版本；
- expected 输出人工核算一次；
- 记录平台官方页面对账截图的哈希或测试说明，截图本身不进入公开仓库。

## 6. 手工验收矩阵

| 维度 | 最低覆盖 |
|---|---|
| macOS | 14、15、当前最新正式版 |
| 芯片 | Apple Silicon；Intel 如声明支持则必须实测 |
| 显示器 | 单屏、双屏、拔插恢复 |
| 外观 | Light、Dark、Auto |
| 权限 | 全允许、拒绝、授权后撤销 |
| 网络 | 正常、离线、慢网、429 |
| 数据 | fresh、stale、empty、error |
| 启动 | 登录启动、手动启动、崩溃后恢复 |

## 7. CI Quality Gates

每次 PR：

- Swift build；
- unit tests；
- adapter contract tests；
- migration tests；
- fixture secret scan；
- formatting/lint；
- Debug 日志敏感模式扫描。
- DesignSystemTokenTests 与 Swiss Ledger 反模式扫描。

Release：

- 全量 UI smoke；
- 性能基线；
- 签名与公证验证；
- 更新 feed 验证；
- 安装/升级/卸载演练；
- 隐私清单审查。

## 8. 完成报告模板

```text
任务：
实现范围：
未实现范围：
测试：
fixture：
性能影响：
隐私检查：
已知限制：
验收结果：PASS / FAIL
```
