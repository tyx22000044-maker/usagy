# 版本控制策略

版本：v1.0  
适用范围：AIUsageMeter 项目全生命周期

## 1. 版本控制概述

### 1.1 版本控制目标

1. **代码管理**：安全、高效地管理代码变更
2. **协作支持**：支持多 Agent 并行开发
3. **变更追踪**：完整记录所有变更历史
4. **发布管理**：支持稳定的版本发布流程
5. **回滚能力**：支持快速回滚到稳定版本

### 1.2 版本控制工具

```text
主要工具：
  - Git：分布式版本控制系统
  - GitHub：代码托管和协作平台
  - GitHub CLI：命令行工具

辅助工具：
  - Git LFS：大文件存储
  - Git Hooks：自动化脚本
  - CI/CD：持续集成/持续部署
```

## 2. 分支策略

### 2.1 分支类型

```text
分支类型：

主分支：
  - main：稳定发布分支
  - develop：开发集成分支

支持分支：
  - feature/*：功能开发分支
  - release/*：发布准备分支
  - hotfix/*：紧急修复分支

Agent 分支：
  - agent-{id}/*：Agent 专用功能分支
```

### 2.2 分支命名规范

```text
分支命名格式：

主分支：
  - main
  - develop

功能分支：
  - agent-{id}/{task-slug}
  - feature/{feature-name}

发布分支：
  - release/{version}
  - release/v1.0.0

修复分支：
  - hotfix/{issue-description}
  - hotfix/fix-crash-on-startup

命名规则：
  - 使用小写字母
  - 使用连字符分隔
  - 避免特殊字符
  - 简洁描述用途
```

### 2.3 分支生命周期

```text
分支生命周期：

1. 创建阶段
   - 从 develop 创建功能分支
   - 设置分支保护规则
   - 配置 CI/CD 触发

2. 开发阶段
   - 在功能分支上开发
   - 定期同步 develop
   - 提交代码变更

3. 集成阶段
   - 创建 Pull Request
   - 代码评审
   - 自动化测试

4. 合并阶段
   - 解决冲突
   - 合并到 develop
   - 删除功能分支

5. 发布阶段
   - 从 develop 创建 release 分支
   - 最终测试和修复
   - 合并到 main 和 develop
```

## 3. 提交规范

### 3.1 提交信息格式

```text
提交信息格式：

<type>(<scope>): <subject>

<body>

<footer>

Type 类型：
  - feat：新功能
  - fix：Bug 修复
  - docs：文档更新
  - style：代码格式
  - refactor：重构
  - test：测试
  - chore：构建/工具

Scope 范围：
  - domain：Domain 层
  - data：Data 层
  - adapter：Adapters 层
  - ui：Features/DesignSystem
  - infra：Infrastructure
  - app：App 层
  - docs：文档
  - ci：CI/CD

Subject 规范：
  - 使用英文
  - 首字母小写
  - 不超过 50 字符
  - 使用祈使语气

Body 规范：
  - 说明修改原因和内容
  - 每行不超过 72 字符
  - 使用中文或英文（保持一致）

Footer 规范：
  - Agent: agent-{id}
  - Task: {task-id}
  - Refs: {issue-link}
```

### 3.2 提交信息示例

```text
示例 1：新功能
feat(adapter): add Claude Code JSONL parser

实现 Claude Code 连接器的基础解析功能：
- 支持 JSONL 格式日志读取
- 实现消息去重逻辑
- 添加基础 fixture 测试

Agent: agent-adapter
Task: T2.1
Refs: #123

示例 2：Bug 修复
fix(domain): correct cumulative delta calculation

修复累计差分计算中的边界条件问题：
- 处理时间戳乱序情况
- 修正负值处理逻辑
- 添加相关测试用例

Agent: agent-arch
Task: T1.3
Refs: #456

示例 3：文档更新
docs(collaboration): update sync schedule

更新协作框架中的同步时间窗口：
- 调整每日同步时间
- 增加周末同步安排
- 更新同步脚本示例

Agent: agent-lead
Task: T0.1
Refs: #789
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
  - 关联相关 Issue

文档更新：
  - 重要文档变更单独提交
  - 小修改可随功能一起提交
  - 保持文档版本同步
```

## 4. 合并策略

### 4.1 合并方式

```text
合并方式：

Squash Merge：
  - 功能分支 → develop
  - 保持历史清晰
  - 适合小功能提交

Merge Commit：
  - develop → main
  - 保留完整历史
  - 适合版本发布

Rebase：
  - 功能分支同步 develop
  - 保持线性历史
  - 避免合并提交
```

### 4.2 合并流程

```text
合并流程：

1. 准备合并
   - 确保代码通过测试
   - 解决所有冲突
   - 更新相关文档

2. 创建 PR
   - 填写 PR 模板
   - 指定评审人
   - 关联相关 Issue

3. 代码评审
   - 评审代码质量
   - 检查测试覆盖
   - 验证功能正确性

4. 自动化检查
   - CI/CD 流水线
   - 代码质量检查
   - 安全扫描

5. 合并代码
   - 选择合并方式
   - 执行合并操作
   - 删除功能分支

6. 后续处理
   - 更新任务状态
   - 通知相关 Agent
   - 监控集成状态
```

### 4.3 冲突解决

```text
冲突解决流程：

1. 冲突检测
   - 自动检测冲突
   - 分析冲突类型
   - 评估影响范围

2. 责任划分
   - 确定责任方
   - 分析冲突原因
   - 制定解决方案

3. 解决冲突
   - 责任方解决冲突
   - 相关 Agent 评审
   - 验证解决方案

4. 合并验证
   - 运行测试
   - 检查功能
   - 确认无回归

5. 记录总结
   - 记录冲突日志
   - 分析根本原因
   - 制定预防措施
```

## 5. 发布管理

### 5.1 版本号规范

```text
版本号格式：
  Major.Minor.Patch

版本号规则：
  - Major：不兼容的 API 修改
  - Minor：向下兼容的功能性新增
  - Patch：向下兼容的问题修正

版本号示例：
  - v1.0.0：首个正式版本
  - v1.1.0：新增功能
  - v1.1.1：Bug 修复
  - v2.0.0：重大更新

预发布版本：
  - v1.0.0-alpha：内部测试版
  - v1.0.0-beta：公开测试版
  - v1.0.0-rc：发布候选版
```

### 5.2 发布流程

```text
发布流程：

1. 发布准备
   - 创建 release 分支
   - 更新版本号
   - 更新变更日志
   - 最终测试

2. 发布测试
   - 完整回归测试
   - 性能测试
   - 安全测试
   - 兼容性测试

3. 发布审批
   - 代码冻结
   - 评审通过
   - 审批发布

4. 正式发布
   - 合并到 main
   - 创建版本标签
   - 生成发布包
   - 发布到分发渠道

5. 发布后
   - 合并回 develop
   - 删除 release 分支
   - 监控发布状态
   - 收集用户反馈
```

### 5.3 热修复流程

```text
热修复流程：

1. 问题识别
   - 紧急问题报告
   - 评估影响范围
   - 确定修复优先级

2. 创建修复
   - 从 main 创建 hotfix 分支
   - 实施修复方案
   - 编写测试用例

3. 测试验证
   - 运行相关测试
   - 验证修复效果
   - 检查无回归

4. 发布修复
   - 合并到 main
   - 创建版本标签
   - 发布修复版本
   - 合并回 develop

5. 问题总结
   - 分析根本原因
   - 制定预防措施
   - 更新相关文档
```

## 6. 标签管理

### 6.1 标签类型

```text
标签类型：

版本标签：
  - v1.0.0：正式版本
  - v1.1.0：功能版本
  - v1.1.1：修复版本

发布标签：
  - release/v1.0.0：发布版本
  - beta/v1.0.0-beta：测试版本

里程碑标签：
  - milestone/phase-1：阶段里程碑
  - milestone/mvp：MVP 里程碑
```

### 6.2 标签命名规范

```text
标签命名格式：
  {type}/{version}

标签命名规则：
  - 使用小写字母
  - 使用连字符分隔
  - 版本号遵循语义化版本
  - 避免特殊字符

标签示例：
  - v1.0.0
  - release/v1.0.0
  - beta/v1.0.0-beta
  - milestone/phase-1
```

### 6.3 标签管理流程

```text
标签管理流程：

1. 创建标签
   - 确定标签类型
   - 选择提交版本
   - 添加标签信息

2. 标签验证
   - 验证标签正确性
   - 检查版本号规范
   - 确认标签位置

3. 标签发布
   - 推送标签到远程
   - 生成发布说明
   - 通知相关 Agent

4. 标签维护
   - 定期清理过期标签
   - 维护标签文档
   - 处理标签问题
```

## 7. 权限管理

### 7.1 分支权限

```text
分支权限：

main 分支：
  - 读取：所有 Agent
  - 写入：Agent-Lead
  - 合并：Agent-Lead
  - 删除：禁止

develop 分支：
  - 读取：所有 Agent
  - 写入：Agent-Lead
  - 合并：所有 Agent（通过 PR）
  - 删除：禁止

功能分支：
  - 读取：所有 Agent
  - 写入：分支所有者
  - 合并：分支所有者
  - 删除：分支所有者
```

### 7.2 代码评审权限

```text
代码评审权限：

评审人设置：
  - Agent-Arch：Domain/Data 层代码
  - Agent-UI：UI 组件代码
  - Agent-Adapter：适配器代码
  - Agent-Lead：跨模块代码

评审要求：
  - 至少一人评审通过
  - 关键模块需多人评审
  - 接口变更需 Agent-Lead 评审

合并权限：
  - 评审通过后可合并
  - Agent-Lead 有最终合并权
  - 紧急情况可跳过评审
```

### 7.3 发布权限

```text
发布权限：

版本发布：
  - Agent-Lead 负责
  - 需要审批流程
  - 需要测试验证

热修复发布：
  - Agent-Lead 负责
  - 可快速通道
  - 需要事后评审

测试发布：
  - 任何 Agent 可发起
  - 需要基本测试
  - 需要标识为测试版
```

## 8. 备份与恢复

### 8.1 备份策略

```text
备份策略：

代码备份：
  - 本地 Git 仓库
  - 远程 GitHub 仓库
  - 定期备份到其他存储

配置备份：
  - Git 配置文件
  - CI/CD 配置
  - 项目配置文件

文档备份：
  - 项目文档
  - 设计文档
  - 变更日志
```

### 8.2 备份频率

```text
备份频率：

实时备份：
  - Git 推送时自动备份
  - CI/CD 触发时备份
  - 重要变更时备份

定期备份：
  - 每日备份代码仓库
  - 每周备份配置文件
  - 每月备份完整项目

手动备份：
  - 重要里程碑前备份
  - 重大变更前备份
  - 发布前备份
```

### 8.3 恢复流程

```text
恢复流程：

1. 问题识别
   - 确定需要恢复的内容
   - 评估恢复优先级
   - 选择恢复时间点

2. 恢复准备
   - 备份当前状态
   - 准备恢复环境
   - 通知相关 Agent

3. 执行恢复
   - 从备份恢复代码
   - 验证恢复结果
   - 解决恢复冲突

4. 恢复验证
   - 运行测试验证
   - 检查功能完整
   - 确认数据一致

5. 后续处理
   - 分析问题原因
   - 制定预防措施
   - 更新备份策略
```

## 9. 工具配置

### 9.1 Git 配置

```text
Git 配置：

全局配置：
  git config --global user.name "Agent-{ID}"
  git config --global user.email "agent-{id}@project.local"
  git config --global core.autocrlf input
  git config --global core.safecrlf warn

项目配置：
  git config user.name "Agent-{ID}"
  git config user.email "agent-{id}@project.local"
  git config core.hooksPath .githooks

别名配置：
  git config alias.st status
  git config alias.co checkout
  git config alias.br branch
  git config alias.ci commit
```

### 9.2 Git Hooks

```text
Git Hooks：

pre-commit：
  - 代码格式检查
  - 静态代码分析
  - 测试运行

commit-msg：
  - 提交信息格式检查
  - 关联 Issue 检查
  - 签名验证

pre-push：
  - 完整测试运行
  - 代码质量检查
  - 安全扫描
```

### 9.3 CI/CD 配置

```text
CI/CD 配置：

构建触发：
  - Push 到 develop
  - Pull Request 创建
  - 标签创建

构建步骤：
  - 代码检出
  - 依赖安装
  - 编译构建
  - 测试运行
  - 代码检查

部署步骤：
  - 构建产物生成
  - 版本标签
  - 发布包创建
  - 部署到测试环境
```

## 10. 监控与报告

### 10.1 监控指标

```text
监控指标：

代码质量：
  - 代码覆盖率
  - 代码复杂度
  - 重复代码率
  - 技术债务

开发效率：
  - 提交频率
  - PR 合并时间
  - 问题解决时间
  - 代码评审时间

协作效果：
  - 冲突发生频率
  - 同步频率
  - 沟通效率
  - 任务完成率
```

### 10.2 报告生成

```text
报告类型：

每日报告：
  - 提交统计
  - 问题汇总
  - 进展更新

每周报告：
  - 里程碑进展
  - 风险识别
  - 下周计划

版本报告：
  - 功能清单
  - 问题修复
  - 性能指标
  - 质量指标
```

### 10.3 问题跟踪

```text
问题跟踪：

问题分类：
  - Bug：代码缺陷
  - Feature：功能需求
  - Task：开发任务
  - Improvement：改进建议

问题优先级：
  - Critical：阻塞性问题
  - High：重要问题
  - Medium：一般问题
  - Low：轻微问题

问题状态：
  - Open：待处理
  - In Progress：处理中
  - Resolved：已解决
  - Closed：已关闭
```