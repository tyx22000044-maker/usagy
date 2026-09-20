# Usagy

> 轻量级 macOS 菜单栏 AI 用量监控工具

[![macOS](https://img.shields.io/badge/macOS-14%2B-blue.svg)](https://www.apple.com/macos/)
[![Swift](https://img.shields.io/badge/Swift-6-orange.svg)](https://swift.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Usagy 是一款常驻 macOS 菜单栏的轻量 AI 用量监控工具，帮助您快速了解各 AI 工具的额度使用情况。无需打开各平台网页，即可在菜单栏或桌面便签中一目了然。

---

## 核心特性

### 一目了然的用量监控

- **3 秒获取答案**：哪个工具最紧张、剩余多少、何时重置
- **菜单栏常驻**：不占用 Dock 位置，随时查看
- **桌面便签**：可选悬浮便签，工作时保持可见

### 多平台支持

首批支持的 AI 工具：
- Claude Code
- Codex
- Qoder（CLI/IDE/Work）
- WorkBuddy
- CodeBuddy
- 千问办公

### 智能提醒

- 额度低于阈值时自动提醒
- 预计耗尽时间预警
- 连接器状态异常通知

### 隐私优先

- 仅读取数值数据，不读取提示词、回复正文或代码
- 所有数据本地存储，不上传云端
- 凭证安全存储于 Keychain

---

## 系统要求

- macOS 14.0 或更高版本
- Apple Silicon 或 Intel Mac

---

## 安装方式

### 方式一：Homebrew（推荐）

```bash
brew install usagy
```

### 方式二：手动安装

1. 从 [Releases](https://github.com/tyx22000044-maker/usagy/releases) 页面下载最新版本的 DMG 文件
2. 打开 DMG 文件，将 Usagy 拖入 Applications 文件夹
3. 首次启动时，系统可能会提示安全警告，请在"系统偏好设置 > 安全性与隐私"中允许运行

### 方式三：从源码构建

```bash
# 克隆仓库
git clone https://github.com/tyx22000044-maker/usagy.git
cd usagy

# 构建项目
swift build

# 运行测试
swift test
```

---

## 快速开始

### 首次启动

1. **启动应用**：Usagy 将出现在菜单栏（右上角）
2. **自动检测**：应用会自动扫描已安装的 AI 工具
3. **授权连接**：根据提示授权需要监控的工具
4. **开始使用**：点击菜单栏图标查看用量信息

### 基本操作

**查看用量概览**
- 点击菜单栏图标，打开 Popover 面板
- 查看各工具的剩余量、使用百分比和重置时间

**开启桌面便签**
- 在 Popover 底部点击"桌面便签"开关
- 便签将显示在桌面上，支持拖动和调整位置

**手动刷新**
- 点击 Popover 顶部的刷新按钮
- 或使用快捷键 `Cmd + R`

---

## 界面说明

### 菜单栏

菜单栏显示最关键的信息：
- 自动模式：显示预计最先耗尽的工具
- 单指标格式：`图标 18%`
- 双指标格式：`Q 18% · C 62%`

### Popover 面板

Popover 面板包含：
- **顶部**：整体状态、最后刷新时间、刷新按钮
- **中部**：各平台卡片列表
- **底部**：桌面便签开关、设置、退出

### 平台卡片

每张卡片显示：
- 平台图标与名称
- 主额度百分比或原生单位
- 重置倒计时
- 新鲜度或错误状态

---

## 配置选项

### 设置菜单

右键点击菜单栏图标，选择"设置"：

- **显示/隐藏平台**：选择要显示的 AI 工具
- **排序方式**：按剩余量、名称或状态排序
- **标题格式**：自动、指定平台或仅图标
- **通知设置**：配置提醒阈值和静音选项
- **开机启动**：是否在登录时自动启动

### 通知配置

默认通知：
- 剩余量低于 10%
- 预计 30 分钟内耗尽
- 连接器连续失败

可选通知：
- 剩余量低于 20%
- 即将重置
- 异常增长

---

## 常见问题

### Q: 为什么某个工具显示"不可用"？

A: 可能的原因：
- 工具未安装或版本不支持
- 缺少必要的文件访问权限
- 数据源格式发生变化

请在设置中查看该工具的诊断信息。

### Q: 数据与官方显示不一致怎么办？

A: 点击数字即可查看数据来源、采集时间、口径和可信等级。如有疑问，请参考"数据解释"功能。

### Q: 如何添加新的 AI 工具支持？

A: 目前支持的工具列表是固定的。未来版本将支持社区贡献的连接器。请关注项目更新。

### Q: 数据会上传到云端吗？

A: 不会。所有数据仅存储在本地，不会上传到任何服务器。

---

## 开发指南

### 项目结构

```text
usagy/
├── Package.swift          # Swift 包管理配置
├── Usagy.xcodeproj        # Xcode 项目文件
├── Sources/               # 源代码
│   ├── App/               # 应用入口
│   ├── Domain/            # 业务领域层
│   ├── Data/              # 数据层
│   ├── Adapters/          # 适配器
│   ├── Infrastructure/    # 基础设施
│   ├── Features/          # 功能模块
│   └── DesignSystem/      # 设计系统
├── Tests/                 # 测试代码
├── Fixtures/              # 测试固件
└── docs/                  # 项目文档
```

### 参与开发

1. Fork 本仓库
2. 创建功能分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -m 'feat: add your feature'`
4. 推送分支：`git push origin feature/your-feature`
5. 创建 Pull Request

### 开发规范

- 遵循 [AGENTS.md](./AGENTS.md) 中的开发规范
- 使用 Swift 6 严格并发模式
- 遵循 Swiss Ledger 设计规范
- 所有变更需通过测试

---

## 文档索引

- [产品规格](./docs/PRODUCT_SPEC.md) - 详细的功能需求和验收标准
- [技术架构](./docs/ARCHITECTURE.md) - 系统架构和模块设计
- [数据模型](./docs/DATA_MODEL.md) - 数据结构和存储设计
- [适配器规范](./docs/ADAPTER_SPEC.md) - 连接器开发规范
- [实施计划](./docs/IMPLEMENTATION_PLAN.md) - 开发计划和里程碑
- [Agent 协作框架](./docs/AGENT_COLLABORATION.md) - 多 Agent 并行开发规范

---

## 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

---

## 联系方式

- 问题反馈：[GitHub Issues](https://github.com/tyx22000044-maker/usagy/issues)
- 功能建议：[GitHub Discussions](https://github.com/tyx22000044-maker/usagy/discussions)

---

## 致谢

感谢以下项目和社区的支持：
- [Swift](https://swift.org/)
- [SwiftUI](https://developer.apple.com/xcode/swiftui/)
- [GRDB](https://github.com/groue/GRDB.swift)

---

**Usagy** - 让 AI 用量一目了然