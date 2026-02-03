# IDD 工作流程指南

> 从想法到实现的 Intent 驱动开发

## 概述

IDD（Intent 驱动开发）遵循一个结构化的工作流程，其中 **Intent 是唯一真相来源**。与传统的代码驱动开发不同，IDD 将人类可读的 Intent 文档置于核心，由 AI 处理实现细节。

```
┌─────────────────────────────────────────────────────────────────────┐
│                         IDD 工作流程                                 │
│                                                                      │
│   定义            细化             构建            维护              │
│                                                                      │
│  ┌──────┐       ┌──────┐        ┌──────┐         ┌──────┐          │
│  │ 评估 │──────▶│ 创建 │───────▶│ 构建 │────────▶│ 同步 │          │
│  └──────┘       └──────┘        └──────┘         └──────┘          │
│                                                                      │
│  IDD 适合吗？   编写 Intent     计划 + TDD      保持同步            │
│  设置项目       审查质量        严格测试        验证健康度          │
│                 批准章节                                            │
└─────────────────────────────────────────────────────────────────────┘
```

## 阶段一：定义

### 步骤 1.1：评估项目适配性

在采用 IDD 之前，评估它是否适合你的项目。

```bash
/intent-assess
```

**适合 IDD 的情况：**
- 新功能或新模块
- 复杂业务逻辑
- 需要团队协作
- 预期长期维护

**不太适合的情况：**
- 快速原型
- 一次性脚本
- 纯机械性重构

参见：[/intent-assess 命令](commands/intent-assess.md)

### 步骤 1.2：初始化 IDD 结构

在项目中设置 Intent 目录结构。

```bash
/intent-init
```

创建的结构：
```
project/
└── intent/
    └── INTENT.md    # 你的第一个 Intent 文件
```

参见：[/intent-init 命令](commands/intent-init.md)

## 阶段二：细化

### 步骤 2.1：创建 Intent

通过引导式访谈将想法转化为结构化的 Intent 文档。

```bash
/intent-interview
```

访谈涵盖：
- **职责**：做什么和不做什么
- **结构**：组件图（ASCII 艺术）
- **API**：函数签名和契约
- **示例**：输入 → 输出 映射

参见：[/intent-interview 命令](commands/intent-interview.md)

### 步骤 2.2：审查设计（可选）

检查 Intent 是否存在过度工程和 YAGNI 违规。

```bash
/intent-critique
```

可以发现：
- 过早抽象
- 不必要的复杂性
- 你不需要的功能

参见：[/intent-critique 命令](commands/intent-critique.md)

### 步骤 2.3：审查和批准

批准 Intent 章节，将它们锁定为实现契约。

```bash
/intent-review
```

章节状态：
- **draft**：进行中，未审查
- **reviewed**：已检查，可能需要修改
- **locked**：已批准，可以实现

参见：[/intent-review 命令](commands/intent-review.md)

### 步骤 2.4：提议变更（按需）

用于类似 PR 体验的协作审查。

```bash
/intent-changes
```

参见：[/intent-changes 命令](commands/intent-changes.md)

## 阶段三：构建

### 步骤 3.1：生成执行计划

将 Intent 转换为结构化、可执行的计划。

```bash
/intent-plan
```

此命令：
1. **验证前置条件**（访谈、评审、依赖）
2. **生成 plan.md** 包含阶段化执行结构
3. **创建 TASK.yaml** 用于执行状态跟踪

计划包含：
- **0 索引的阶段** 带有明确交付物
- **6 类测试** 每阶段（Happy/Bad/Edge/Security/Data Leak/Data Damage）
- **E2E Gate** 每阶段的验证脚本
- **复选框跟踪** 用于进度追踪

参见：[/intent-plan 命令](commands/intent-plan.md)

### 步骤 3.2：TDD 执行

有了计划后，使用严格的 TDD 纪律执行。

```bash
/intent-build-now    # 直接执行
# 或
/swarm run           # 使用 TaskSwarm（多 Agent）
```

```
┌─────────────────────────────────────────────────────────────┐
│                    执行流程                                   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  读取 plan.md                                       │    │
│  │  - 阶段、测试、验收标准                             │    │
│  └─────────────────────┬───────────────────────────────┘    │
│                        ▼                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  对于每个阶段：                                      │    │
│  │  1. 先写所有测试（6 类）                            │    │
│  │  2. 运行测试（预期失败）                            │    │
│  │  3. 实现代码                                        │    │
│  │  4. 运行测试（预期通过）                            │    │
│  │  5. 运行 E2E Gate 验证                              │    │
│  │  6. 更新复选框，提交                                │    │
│  └─────────────────────┬───────────────────────────────┘    │
│                        ▼                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  所有阶段完成                                        │    │
│  │  - 最终 E2E 验证                                    │    │
│  │  - TASK.yaml 状态: review                           │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**测试类别（全部 6 类必需）：**

| 类别 | 目的 |
|------|------|
| Happy Path | 正常预期用法 |
| Bad Path | 无效输入、错误条件 |
| Edge Cases | 边界条件 |
| Security | 漏洞防护 |
| Data Leak | 信息泄露防护 |
| Data Damage | 数据完整性保护 |

参见：[/intent-build-now 命令](commands/intent-build-now.md)

## 阶段四：维护

### 步骤 4.1：同步实现细节

实现完成后，将确认的细节同步回 Intent。

```bash
/intent-sync
```

捕获的内容：
- 最终 API 签名
- 实际数据结构
- 实现决策

参见：[/intent-sync 命令](commands/intent-sync.md)

### 步骤 4.2：验证一致性

运行检查以确保 Intent 和实现保持一致。

```bash
/intent-check
```

这会自动触发验证 Agent。

参见：[/intent-check 命令](commands/intent-check.md)

### 步骤 4.3：生成报告

从 Intent 文件创建人类可读的文档。

```bash
/intent-report
```

参见：[/intent-report 命令](commands/intent-report.md)

### 步骤 4.4：分享经验（可选）

为他人记录你的 IDD 之旅。

```bash
/intent-story
```

参见：[/intent-story 命令](commands/intent-story.md)

## 快速参考

### 典型的新功能流程

```bash
/intent-interview     # 从你的想法创建 Intent
/intent-critique      # 检查是否过度工程
/intent-review        # 批准关键章节
/intent-build-now     # 验证并开始 TDD 执行
/intent-sync          # 完成后同步回来
```

### 现有项目采用

```bash
/intent-assess        # 评估适配性
/intent-init          # 设置结构
/intent-interview     # 记录现有模块
```

### 健康检查

```bash
/intent-check         # 运行所有验证
```

## 下一步

- [命令参考](commands/README.md) - 每个命令的详细文档
- [常见问题](faq.md) - 常见问题与解答
- [Intent 标准](intent-standard.md) - 文件格式规范
- [TaskSwarm](https://github.com/ArcBlock/teamswarm) - 自主任务执行系统
