# 开发指南

版本：v1.0  
适用对象：所有开发 Agent

## 1. 开发环境设置

### 1.1 环境要求

```text
开发环境要求：
  - macOS 14.0+
  - Xcode 16.0+
  - Swift 6.0+
  - Git 2.30+
  - GitHub CLI (gh) 2.0+（可选）
```

### 1.2 项目初始化

```bash
# 1. 克隆仓库
git clone https://github.com/your-org/AIUsageMeter.git
cd AIUsageMeter

# 2. 安装依赖（如有）
swift package resolve

# 3. 打开项目
open AIUsageMeter.xcodeproj

# 4. 验证环境
swift build
swift test
```

### 1.3 Agent 身份配置

每个 Agent 必须配置 Git 身份：

```bash
# 设置 Agent 身份
git config user.name "Agent-{ID}"
git config user.email "agent-{id}@project.local"

# 示例：
git config user.name "Agent-Arch"
git config user.email "agent-arch@project.local"
```

## 2. 分支管理规范

### 2.1 分支命名规则

```text
分支命名格式：
  agent-{id}/{task-slug}

命名规则：
  - 使用小写字母和连字符
  - task-slug 简洁描述任务内容
  - 不超过 50 字符
  - 避免使用特殊字符

示例：
  agent-arch/domain-usage-event
  agent-arch/sqlite-ledger-migration
  agent-ui/menu-bar-popover
  agent-ui/floating-card-component
  agent-adapter/claude-jsonl-parser
  agent-adapter/codex-adapter
  agent-lead/adapter-protocol-v2
```

### 2.2 分支生命周期管理

```text
分支生命周期：

1. 创建分支
   git checkout develop
   git pull origin develop
   git checkout -b agent-{id}/{task}

2. 开发过程中定期同步
   git fetch origin
   git rebase origin/develop

3. 完成开发后推送
   git push origin agent-{id}/{task} --force-with-lease

4. 创建 Pull Request
   gh pr create --base develop --head agent-{id}/{task}

5. 代码评审通过后合并
   gh pr merge --squash

6. 删除本地和远程分支
   git branch -d agent-{id}/{task}
   git push origin --delete agent-{id}/{task}
```

### 2.3 分支保护规则

```text
分支保护策略：

main 分支：
  - 禁止直接推送
  - 需要 PR 审核
  - 需要 CI 通过
  - 禁止 force push

develop 分支：
  - 禁止直接推送
  - 需要 PR 审核
  - 需要 CI 通过
  - 允许 force push（仅限 rebase）

功能分支：
  - 允许直接推送
  - 建议定期 rebase develop
```

## 3. 代码提交规范

### 3.1 Commit 信息格式

```text
<type>(<scope>): <subject>

<body>

<footer>
```

**Type 类型：**

| Type | 说明 | 示例 |
|------|------|------|
| feat | 新功能 | feat(adapter): add Claude Code parser |
| fix | Bug 修复 | fix(domain): correct token calculation |
| docs | 文档更新 | docs(guide): update commit conventions |
| style | 代码格式 | style(ui): fix indentation |
| refactor | 重构 | refactor(data): extract repository |
| test | 测试 | test(adapter): add Claude fixtures |
| chore | 构建/工具 | chore: update Swift version |

**Scope 范围：**

| Scope | 说明 |
|-------|------|
| domain | Domain 层 |
| data | Data 层 |
| adapter | Adapters 层 |
| ui | Features/DesignSystem |
| infra | Infrastructure |
| app | App 层 |
| docs | 文档 |
| ci | CI/CD |

**Subject 规范：**

- 使用英文
- 首字母小写
- 不超过 50 字符
- 使用祈使语气

**Body 规范：**

- 说明修改原因和内容
- 每行不超过 72 字符
- 使用中文或英文（保持一致）

**Footer 规范：**

```text
Agent: agent-{id}
Task: {task-id}
Refs: {issue-link}
```

### 3.2 Commit 信息示例

```text
feat(adapter): add Claude Code JSONL parser

实现 Claude Code 连接器的基础解析功能：
- 支持 JSONL 格式日志读取
- 实现消息去重逻辑
- 添加基础 fixture 测试

Agent: agent-adapter
Task: T2.1
Refs: #123
```

```text
fix(domain): correct cumulative delta calculation

修复累计差分计算中的边界条件问题：
- 处理时间戳乱序情况
- 修正负值处理逻辑
- 添加相关测试用例

Agent: agent-arch
Task: T1.3
Refs: #456
```

### 3.3 提交频率

```text
提交频率建议：

功能开发：
  - 每完成一个独立功能点提交一次
  - 每日至少提交一次进展
  - 避免积累大量修改后一次性提交

Bug 修复：
  - 修复完成后立即提交
  - 包含测试用例

文档更新：
  - 重要文档变更单独提交
  - 小修改可随功能一起提交
```

## 4. 代码评审规范

### 4.1 评审流程

```text
代码评审流程：

1. 准备评审
   - 确保代码通过本地测试
   - 更新相关文档
   - 添加必要的测试用例

2. 创建 PR
   - 填写 PR 模板
   - 关联相关 Issue
   - 指定评审人

3. 评审过程
   - 评审人检查代码质量
   - 提出修改建议
   - 讨论技术方案

4. 修改完善
   - 根据评审意见修改
   - 重新提交代码
   - 请求再次评审

5. 合并代码
   - 评审通过后合并
   - 删除功能分支
   - 更新任务状态
```

### 4.2 评审检查清单

```text
代码评审检查清单：

代码质量：
  □ 代码风格符合项目规范
  □ 无冗余代码和死代码
  □ 命名清晰易懂
  □ 注释适当且准确

功能正确性：
  □ 实现符合需求规格
  □ 边界条件处理正确
  □ 错误处理完善
  □ 性能符合预期

测试覆盖：
  □ 单元测试完整
  □ 测试用例覆盖边界情况
  □ 测试可重复执行
  □ 测试文档清晰

文档更新：
  □ 相关文档已更新
  □ 接口文档准确
  □ 变更日志已记录

安全隐私：
  □ 无敏感信息泄露
  □ 符合隐私规范
  □ 权限处理正确
```

### 4.3 评审责任

```text
评审责任分配：

Agent-Arch：
  - 评审 Domain/Data 层代码
  - 评审接口设计合理性
  - 评审并发和性能相关代码

Agent-UI：
  - 评审 UI 组件代码
  - 评审设计规范符合性
  - 评审无障碍实现

Agent-Adapter：
  - 评审适配器实现
  - 评审 fixture 质量
  - 评审隐私合规性

Agent-Lead：
  - 评审跨模块接口
  - 评审集成测试
  - 评审文档完整性
```

## 5. 测试规范

### 5.1 测试类型

```text
测试类型：

单元测试：
  - 测试单个函数或方法
  - 隔离外部依赖
  - 快速执行

集成测试：
  - 测试模块间交互
  - 使用真实接口
  - 验证数据流

契约测试：
  - 测试接口实现
  - 验证协议符合性
  - 使用 fixture 驱动

UI 测试：
  - 测试用户界面
  - 验证交互流程
  - 检查无障碍功能
```

### 5.2 测试命名规范

```text
测试命名格式：
  test_{功能}_{场景}_{预期结果}

示例：
  test_usageEvent_creation_withValidData_succeeds
  test_usageEvent_creation_withInvalidData_throwsError
  test_tokenCalculation_withMixedUnits_returnsCorrectTotal
  test_adapter_detect_withValidInstallation_returnsDetected
```

### 5.3 测试覆盖率要求

```text
测试覆盖率要求：

Domain 层：
  - 单元测试覆盖率 ≥ 90%
  - 核心业务逻辑 100% 覆盖

Data 层：
  - 单元测试覆盖率 ≥ 80%
  - 数据库操作 100% 覆盖

Adapters 层：
  - 契约测试 100% 覆盖
  - 边界条件测试完整

Features 层：
  - 关键路径测试覆盖
  - 无障碍测试覆盖
```

### 5.4 测试执行

```bash
# 运行所有测试
swift test

# 运行特定模块测试
swift test --filter DomainTests
swift test --filter DataTests
swift test --filter AdapterContractTests

# 运行特定测试用例
swift test --filter test_usageEvent_creation

# 生成测试覆盖率报告
swift test --enable-code-coverage
```

## 6. 文档规范

### 6.1 文档类型

```text
文档类型：

设计文档：
  - 产品规格 (PRODUCT_SPEC.md)
  - 技术架构 (ARCHITECTURE.md)
  - 设计规范 (DESIGN_SPEC.md)
  - 数据模型 (DATA_MODEL.md)

开发文档：
  - 实施计划 (IMPLEMENTATION_PLAN.md)
  - 测试计划 (TEST_PLAN.md)
  - 开发指南 (DEVELOPMENT_GUIDE.md)
  - 协作框架 (AGENT_COLLABORATION.md)

接口文档：
  - 适配器规范 (ADAPTER_SPEC.md)
  - 接口定义 (Domain/Protocols/)
  - API 文档 (自动生成)

维护文档：
  - 决策记录 (DECISIONS.md)
  - 变更日志 (CHANGELOG.md)
  - 故障排除 (TROUBLESHOOTING.md)
```

### 6.2 文档更新流程

```text
文档更新流程：

1. 识别变更
   - 代码变更影响文档
   - 需求变更需要更新
   - 发现文档错误

2. 更新文档
   - 修改相关文档
   - 保持格式一致
   - 更新版本号

3. 评审文档
   - 相关 Agent 评审
   - 确保准确性
   - 检查完整性

4. 提交变更
   - 单独提交文档变更
   - 关联相关代码变更
   - 更新变更日志
```

### 6.3 文档模板

```text
文档模板结构：

# 标题

版本：v1.0
最后更新：YYYY-MM-DD
负责人：Agent-{id}

## 1. 概述
[简要描述文档目的和范围]

## 2. 详细内容
[主要内容]

## 3. 示例
[使用示例]

## 4. 参考
[相关文档和链接]

## 5. 变更历史
[版本变更记录]
```

## 7. 问题处理流程

### 7.1 问题分类

```text
问题分类：

技术问题：
  - 编译错误
  - 运行时错误
  - 性能问题
  - 兼容性问题

协作问题：
  - 接口冲突
  - 依赖问题
  - 沟通障碍
  - 进度延迟

需求问题：
  - 需求不明确
  - 需求变更
  - 需求冲突
  - 需求遗漏
```

### 7.2 问题处理流程

```text
问题处理流程：

1. 问题识别
   - 发现问题及时记录
   - 评估问题影响范围
   - 确定问题优先级

2. 问题分析
   - 收集相关信息
   - 分析问题原因
   - 评估解决方案

3. 问题解决
   - 实施解决方案
   - 验证解决效果
   - 更新相关文档

4. 问题预防
   - 分析根本原因
   - 制定预防措施
   - 更新流程规范
```

### 7.3 问题升级机制

```text
问题升级路径：

Level 1：Agent 自行解决
  - 问题在 Agent 能力范围内
  - 解决方案明确
  - 影响范围有限

Level 2：协作 Agent 协商
  - 问题涉及多个模块
  - 需要技术方案讨论
  - 影响开发进度

Level 3：Agent-Lead 仲裁
  - 问题无法达成一致
  - 涉及架构决策
  - 影响项目方向

Level 4：用户决策
  - 重大技术选型
  - 需求变更确认
  - 项目范围调整
```

## 8. 沟通协作规范

### 8.1 沟通渠道

```text
沟通渠道：

日常沟通：
  - Pull Request 评论
  - Issue 讨论
  - 文档协作

技术讨论：
  - 技术方案评审
  - 接口设计讨论
  - 问题排查

进度同步：
  - 每日站会
  - 周报总结
  - 里程碑评审

紧急沟通：
  - 直接通知
  - 电话/视频
  - 紧急 Issue
```

### 8.2 沟通规范

```text
沟通规范：

信息传递：
  - 信息清晰明确
  - 避免歧义
  - 提供必要上下文

反馈及时性：
  - 重要信息 24 小时内响应
  - 普通信息 48 小时内响应
  - 紧急问题立即响应

文档记录：
  - 重要决策记录文档
  - 技术讨论形成文档
  - 问题解决方案文档化
```

### 8.3 会议规范

```text
会议规范：

每日站会：
  - 时间：每日 09:00
  - 时长：15 分钟
  - 内容：昨日进展、今日计划、阻塞问题

周计划会：
  - 时间：每周一 10:00
  - 时长：30 分钟
  - 内容：周计划、依赖确认、风险识别

周总结会：
  - 时间：每周五 16:00
  - 时长：30 分钟
  - 内容：周总结、问题分析、下周计划
```

## 9. 工具使用规范

### 9.1 Git 工具

```text
Git 使用规范：

提交信息：
  - 遵循提交规范
  - 使用英文或中文（保持一致）
  - 信息简洁明了

分支管理：
  - 遵循分支命名规范
  - 定期同步 develop
  - 及时删除已合并分支

代码合并：
  - 使用 Squash Merge
  - 解决冲突后合并
  - 保持历史清晰
```

### 9.2 开发工具

```text
开发工具：

IDE：
  - Xcode 16.0+
  - 安装必要插件
  - 配置代码格式化

构建工具：
  - Swift Package Manager
  - Xcode 构建系统
  - CI/CD 流水线

测试工具：
  - XCTest 框架
  - 代码覆盖率工具
  - 性能测试工具
```

### 9.3 协作工具

```text
协作工具：

代码托管：
  - GitHub
  - Pull Request
  - Issue 跟踪

文档协作：
  - Markdown 文档
  - 版本控制
  - 在线协作

项目管理：
  - 任务跟踪
  - 进度管理
  - 风险管理
```

## 10. 最佳实践

### 10.1 编码最佳实践

```text
编码最佳实践：

代码风格：
  - 遵循 Swift API 设计指南
  - 使用一致的命名规范
  - 保持代码简洁清晰

错误处理：
  - 使用 typed error
  - 提供有意义的错误信息
  - 适当处理边界条件

性能优化：
  - 避免不必要的内存分配
  - 使用适当的数据结构
  - 优化热点代码路径
```

### 10.2 协作最佳实践

```text
协作最佳实践：

沟通协作：
  - 及时沟通问题
  - 主动分享知识
  - 尊重他人意见

代码共享：
  - 遵循接口规范
  - 提供清晰文档
  - 保持向后兼容

团队协作：
  - 支持团队成员
  - 分享最佳实践
  - 共同解决问题
```

### 10.3 持续改进

```text
持续改进：

定期回顾：
  - 每周回顾开发流程
  - 分析问题根因
  - 制定改进措施

知识分享：
  - 技术分享会
  - 文档完善
  - 最佳实践推广

流程优化：
  - 识别瓶颈环节
  - 优化工作流程
  - 提高开发效率
```