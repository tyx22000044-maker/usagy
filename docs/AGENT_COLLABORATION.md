# Agent 并行开发协作框架

版本：v1.0  
适用阶段：Phase 1–6 并行开发期

## 1. 协作架构概述

### 1.1 Agent 角色定义

本项目采用 4 Agent 并行开发模式：

| Agent ID | 角色名称 | 主要职责 | 核心模块 |
|----------|----------|----------|----------|
| Agent-Arch | 架构 Agent | 核心框架、Domain 模型、数据层 | Domain/, Data/, Infrastructure/ |
| Agent-UI | UI Agent | 用户界面、设计系统、交互 | Features/, DesignSystem/ |
| Agent-Adapter | 适配器 Agent | 连接器实现、数据采集 | Adapters/ |
| Agent-Lead | 主控 Agent | 协议定义、集成测试、发布 | 协议、集成、验收 |

### 1.2 并行开发原则

1. **协议优先**：所有共享接口必须先由 Agent-Lead 定义并锁定
2. **模块隔离**：每个 Agent 负责独立的文件目录，最小化代码重叠
3. **Fixture 驱动**：适配器开发必须先提交脱敏 fixture
4. **增量集成**：定期同步，避免长时间分叉导致大规模冲突

## 2. 任务分配机制

### 2.1 模块划分与职责边界

```text
AIUsageMeter/
├── Sources/
│   ├── Domain/           # Agent-Arch 独占
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Protocols/    # 接口定义，Agent-Lead 审核
│   │   └── Policies/
│   ├── Data/             # Agent-Arch 独占
│   │   ├── Database/
│   │   ├── Repositories/
│   │   ├── Migrations/
│   │   └── Queries/
│   ├── Infrastructure/   # Agent-Arch 独占
│   ├── Adapters/         # Agent-Adapter 独占
│   │   ├── Shared/       # 接口协议，Agent-Lead 定义
│   │   ├── ClaudeCode/
│   │   ├── Codex/
│   │   └── ...
│   ├── Features/         # Agent-UI 独占
│   │   ├── MenuBar/
│   │   ├── FloatingCard/
│   │   ├── Detail/
│   │   ├── Connectors/
│   │   └── Settings/
│   ├── DesignSystem/     # Agent-UI 独占
│   └── App/              # Agent-Lead 独占
├── Tests/                # 按模块分配
├── Fixtures/             # Agent-Adapter 主导
└── docs/                 # Agent-Lead 维护
```

### 2.2 任务依赖矩阵

```text
Phase 0 (基础)：
  Agent-Lead → 定义所有协议接口
  Agent-Arch → 工程骨架、Domain 模型
  Agent-UI   → DesignSystem 基础组件

Phase 1 (数据内核)：
  Agent-Arch → SQLite Ledger、Ingestion、Snapshot
  Agent-UI   → 空态 UI、加载态
  Agent-Adapter → fixture 工具链

Phase 2 (首个切片)：
  Agent-Arch → Snapshot 服务
  Agent-UI   → MenuBar UI
  Agent-Adapter → Claude/Codex 适配器

Phase 3 (扩展)：
  Agent-Arch → 通知引擎
  Agent-UI   → FloatingCard、Detail
  Agent-Adapter → 国内连接器（并行）

Phase 4 (收尾)：
  Agent-Lead → 集成测试、发布
  Agent-Arch → 性能优化
  Agent-UI   → 无障碍、深色模式
  Agent-Adapter → 连接器诊断
```

## 3. 代码管理策略

### 3.1 分支命名规范

```text
main                              # 稳定发布分支
├── develop                       # 开发集成分支
│   ├── agent-arch/domain-models  # Agent-Arch 功能分支
│   ├── agent-arch/sqlite-ledger
│   ├── agent-ui/menu-bar         # Agent-UI 功能分支
│   ├── agent-ui/floating-card
│   ├── agent-adapter/claude      # Agent-Adapter 功能分支
│   ├── agent-adapter/codex
│   └── agent-lead/protocol-v2    # Agent-Lead 功能分支
```

分支命名格式：`agent-{id}/{task-slug}`

示例：
- `agent-arch/domain-usage-event`
- `agent-ui/swiss-ledger-tokens`
- `agent-adapter/claude-jsonl-parser`
- `agent-lead/adapter-protocol-v2`

### 3.2 分支生命周期

1. **创建**：从 `develop` 创建功能分支
2. **开发**：在功能分支上提交代码
3. **同步**：每日从 `develop` rebase/merge
4. **评审**：完成开发后创建 PR
5. **合并**：Agent-Lead 审核后合并到 `develop`
6. **删除**：合并后删除功能分支

### 3.3 合并策略

- **功能分支 → develop**：Squash Merge（保持历史清晰）
- **develop → main**：Merge Commit（保留完整历史）
- **禁止**：直接向 `main` 或 `develop` 推送代码

## 4. 冲突预防措施

### 4.1 代码所有权划分

```text
# .github/CODEOWNERS 或等效配置

# Agent-Arch 独占区域
/Sources/Domain/        @agent-arch
/Sources/Data/          @agent-arch
/Sources/Infrastructure/ @agent-arch

# Agent-UI 独占区域
/Sources/Features/      @agent-ui
/Sources/DesignSystem/  @agent-ui

# Agent-Adapter 独占区域
/Sources/Adapters/      @agent-adapter
/Fixtures/              @agent-adapter

# Agent-Lead 独占区域
/Sources/App/           @agent-lead
/docs/                  @agent-lead

# 共享区域（需双人审核）
/Sources/Domain/Protocols/ @agent-arch @agent-lead
/Sources/Adapters/Shared/  @agent-adapter @agent-lead
```

### 4.2 接口锁定协议

所有跨模块接口必须遵循以下流程：

1. **提议**：任何 Agent 可提议接口变更
2. **设计**：Agent-Lead 主导接口设计文档
3. **评审**：相关 Agent 评审接口设计
4. **锁定**：Agent-Lead 批准后标记为 `LOCKED`
5. **变更**：锁定后变更需走正式 RFC 流程

接口锁定标记示例：

```swift
// MARK: - LOCKED Interface v1.0
// Approved by: Agent-Lead
// Date: 2026-09-20
// Change requires: RFC-XXX
protocol UsageAdapter: Sendable {
    // ...
}
```

### 4.3 文件级冲突预防

| 文件类型 | 预防策略 |
|----------|----------|
| Package.swift | Agent-Lead 独占修改 |
| *.xcodeproj | Agent-Lead 独占修改 |
| Domain/Protocols/* | 接口锁定，变更需 RFC |
| Adapters/Shared/* | 接口锁定，变更需 RFC |
| 其他模块文件 | 按 CODEOWNERS 划分 |

## 5. 同步机制

### 5.1 同步时间窗口

```text
每日同步流程：
  09:00  各 Agent 从 develop pull/rebase
  12:00  提交当日进展到功能分支
  18:00  创建 PR 到 develop
  19:00  Agent-Lead 审核并合并

每周同步流程：
  周一 10:00  周计划会议，确认依赖和接口
  周五 16:00  周总结，更新协作矩阵
```

### 5.2 同步频率要求

| 任务类型 | 最大同步间隔 | 同步方式 |
|----------|--------------|----------|
| 接口定义变更 | 立即 | 通知所有相关 Agent |
| 功能开发 | 每日 | rebase develop |
| Bug 修复 | 每日 | rebase develop |
| 文档更新 | 每周 | 直接提交 |

### 5.3 同步操作规范

```bash
# 每日同步脚本模板
#!/bin/bash

# 1. 保存当前工作
git stash

# 2. 切换到 develop 并拉取最新
git checkout develop
git pull origin develop

# 3. 切换回功能分支并 rebase
git checkout agent-{id}/{task}
git rebase develop

# 4. 解决冲突（如有）
# ...

# 5. 恢复工作
git stash pop

# 6. 推送更新
git push origin agent-{id}/{task} --force-with-lease
```

## 6. 冲突解决预案

### 6.1 冲突检测

**自动检测触发条件：**

1. PR 合并时检测到文件冲突
2. rebase 时检测到内容冲突
3. 接口定义文件被多个 Agent 修改

**冲突分类：**

| 冲突类型 | 严重程度 | 解决优先级 |
|----------|----------|------------|
| 接口定义冲突 | 高 | 立即解决 |
| 模块边界冲突 | 中 | 24 小时内解决 |
| 实现细节冲突 | 低 | 48 小时内解决 |

### 6.2 责任划分

```text
冲突责任判定规则：

1. 后提交者负责解决冲突
2. 接口变更冲突由变更提出方负责
3. 多人同时修改同一文件，由 Agent-Lead 仲裁

责任分配示例：
  Agent-Arch 修改了 Domain/Models/UsageEvent.swift
  Agent-UI 也修改了同一文件
  → Agent-UI 负责解决冲突（后提交）
```

### 6.3 解决方案评审

冲突解决流程：

1. **识别**：检测冲突类型和影响范围
2. **分析**：确定冲突原因和责任方
3. **方案**：责任方提出解决方案
4. **评审**：相关 Agent 评审方案
5. **实施**：执行解决方案
6. **验证**：运行测试确保无回归
7. **记录**：更新冲突日志

### 6.4 合并验证

冲突解决后必须通过以下验证：

```text
验证清单：
  □ 编译通过（swift build）
  □ 单元测试通过（swift test）
  □ 接口兼容性检查
  □ 无新增警告
  □ 代码风格检查
  □ 文档更新（如需要）
```

## 7. 提交规范

### 7.1 Commit 信息格式

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
| docs | 文档更新 | docs(collaboration): update sync schedule |
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

### 7.2 Commit 信息示例

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

### 7.3 PR 标题格式

```text
[Agent-ID] Type(scope): description

示例：
[Agent-Adapter] feat(adapter): implement Claude Code parser
[Agent-Arch] fix(domain): fix token calculation edge case
[Agent-UI] feat(ui): add floating card component
```

## 8. 协作文档

### 8.1 Agent 协作矩阵

```text
协作依赖矩阵：

Agent-Arch ←→ Agent-Lead
  - 接口定义审批
  - 集成测试协调
  - 性能基准确认

Agent-Arch ←→ Agent-UI
  - ViewModel 接口
  - 数据模型映射
  - 状态流定义

Agent-Arch ←→ Agent-Adapter
  - 数据库 Schema
  - 事件模型定义
  - 仓储接口

Agent-UI ←→ Agent-Lead
  - 设计规范审核
  - 无障碍标准
  - 交互流程确认

Agent-UI ←→ Agent-Adapter
  - 适配器状态展示
  - 错误信息格式
  - 能力映射

Agent-Adapter ←→ Agent-Lead
  - 连接器发布门槛
  - Fixture 审核
  - 隐私合规检查
```

### 8.2 接口规范文档

所有跨模块接口必须在以下位置定义：

```text
接口规范位置：
  /Sources/Domain/Protocols/     # 核心业务接口
  /Sources/Adapters/Shared/      # 适配器接口
  /docs/interfaces/              # 接口设计文档（如需要）
```

接口定义模板：

```swift
// MARK: - 接口名称
// 版本: v1.0
// 状态: LOCKED / DRAFT
// 负责人: Agent-{id}
// 最后更新: YYYY-MM-DD
// 变更历史:
//   - v1.0 (YYYY-MM-DD): 初始定义

protocol InterfaceName: Sendable {
    // 接口方法
}
```

### 8.3 沟通渠道

| 沟通类型 | 渠道 | 频率 |
|----------|------|------|
| 接口变更 | PR 评审 | 实时 |
| 进展同步 | 每日站会 | 每日 |
| 技术讨论 | Issue/Discussion | 按需 |
| 紧急问题 | 直接通知 | 立即 |

### 8.4 依赖关系图

```text
开发依赖关系：

Phase 0:
  Agent-Lead (协议定义)
      ↓
  Agent-Arch (工程骨架)
      ↓
  Agent-UI (DesignSystem)

Phase 1:
  Agent-Arch (Domain/Data)
      ↓
  Agent-Adapter (Fixture 工具链)
      ↓
  Agent-UI (空态 UI)

Phase 2:
  Agent-Arch (Snapshot 服务)
      ↓
  Agent-Adapter (Claude/Codex)
      ↓
  Agent-UI (MenuBar UI)

Phase 3:
  Agent-Arch (通知引擎)
      ↓
  Agent-Adapter (国内连接器)
      ↓
  Agent-UI (FloatingCard)
```

## 9. 执行指南

### 9.1 Agent 启动检查清单

每个 Agent 开始工作前必须：

```text
启动检查清单：
  □ 阅读 AGENTS.md
  □ 阅读本协作框架文档
  □ 确认当前 Phase 和任务
  □ 检查接口锁定状态
  □ 从 develop 创建功能分支
  □ 同步最新代码
```

### 9.2 日常工作流程

```text
日常工作流程：

09:00  检查同步状态，pull 最新 develop
09:30  开始开发任务
12:00  提交进展到功能分支
14:00  继续开发
18:00  创建 PR，请求评审
18:30  更新任务状态
```

### 9.3 问题升级流程

```text
问题升级路径：

Level 1: Agent 自行解决
  ↓ (无法解决)
Level 2: 协作 Agent 协商
  ↓ (无法达成一致)
Level 3: Agent-Lead 仲裁
  ↓ (重大分歧)
Level 4: 用户决策
```

## 10. 附录

### 10.1 常用命令

```bash
# 创建功能分支
git checkout develop
git pull origin develop
git checkout -b agent-{id}/{task}

# 同步 develop
git fetch origin
git rebase origin/develop

# 推送功能分支
git push origin agent-{id}/{task} --force-with-lease

# 创建 PR
gh pr create --base develop --head agent-{id}/{task}
```

### 10.2 冲突日志模板

```text
冲突日志：

日期: YYYY-MM-DD
冲突类型: [接口/模块/实现]
涉及 Agent: [Agent-1, Agent-2]
冲突文件: [file1.swift, file2.swift]
原因: [简述原因]
解决方案: [简述方案]
责任人: [Agent-ID]
解决时间: [HH:MM]
验证结果: [通过/失败]
```

### 10.3 接口变更 RFC 模板

```text
RFC-XXX: 接口变更标题

状态: [DRAFT/REVIEW/APPROVED/REJECTED]
作者: Agent-{id}
日期: YYYY-MM-DD

## 动机
[为什么需要变更]

## 方案
[具体变更内容]

## 影响
[影响范围和兼容性]

## 替代方案
[考虑过的其他方案]

## 决策
[最终决策和理由]
```