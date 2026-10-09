# Kilo AI Agent 工程哲学（材料笔记）

别名：Kilo、Kilo AI、Kilo Code。

> Kilo 是一个开源 AI 编码 Agent 平台。核心洞察：**可审查性是 AI 工程的瓶颈**——如果输出无法被审查，就无法被信任，也就不会被合并。

## 基本信息

- 创始人：Scott Breitenother（CEO，前 Brooklyn Data 创始人）+ Sid Sijbrandij（GitLab 联合创始人）
- 融资：2025 年 12 月完成 800 万美元种子轮（Cota Capital 领投）
- 规模：截至 2026 年初，超过 300 万下载量，处理超过 40 万亿 token

## 核心原则：任务粒度 = 审查能力

> 任务大小，应该和你的审查能力匹配。如果一个人没法在一次坐下来就评估完 agent 的输出，那这个任务就拆得太大了。

这是 Kilo 设计的首要原则：**任务规模应以人类单次审查可完成的范围内为上限**。

### 推论

1. **任务粒度 = f(审查者经验)**：新手拆更小，专家可拆更大
2. **"太大"是分解问题，不是能力问题**：如果审查者感到吃力，说明任务需要进一步拆分
3. **可操作规则**：每个子任务输出应在 15-30 分钟内可完整评估

## 多 Agent 架构

Kilo 采用四层 Agent 架构，形成从设计到验证的流水线：

```
Architect（架构）→ Orchestrator（编排）→ Code Agent（编码）→ Debug Agent（调试）
```

| 层级 | 职责 | 说明 |
|------|------|------|
| **Architect Agent** | 架构设计 | 编码前规划文件结构和系统设计 |
| **Orchestrator Agent** | 任务分解 | 将大功能拆解为可审查的小单元，分配并行工作流 |
| **Code Agent** | 编码实现 | 执行具体的代码编写 |
| **Debug Agent** | 调试修复 | 故障排查，可自主修复执行错误 |

### 并行执行

Orchestrator 可将一个功能拆为多个并行工作流（如：API 端点 + 测试套件 + 文档），每个由独立 Agent 处理，各自拥有明确的指令和边界。

## 可审查性体系

### 内置审查工具

- **AI Code Reviewer**：自动扫描 PR 中的 bug、风格问题、安全漏洞
- 审查风格可配置：严格/平衡/宽松
- 审查范围：安全漏洞、性能问题、bug 检测、代码风格、测试覆盖、文档

### 权限控制系统

每个 Agent 的权限可细粒度控制：

| 级别 | 行为 |
|------|------|
| allow | 自动执行 |
| ask | 需用户确认 |
| deny | 禁止执行 |

支持设置**只读审查 Agent**——可分析代码但不允许修改。

### 信任建立路径

Kilo 推荐的团队信任演进路径：

```
聊天式 AI 辅助 → 单 Agent → 多 Agent → 自动化代码审查
```

每一步扩大信任范围，核心前提是每一步的输出都保持在可审查的粒度内。

## 开发者角色转变

> "你的工作从'写每一行代码'变成'设计循环'——你决定任务的边界、模型、权限、环境和验证步骤。Agent 写代码，你决定代码是否应该存在。" — Brendan O'Leary（Kilo）

完整的工程循环：**plan → scope → run → verify → review → merge**

## 概念关联

- [问题分解](<../../教育/学习方法/问题分解.md>) — 提供通用分解原则，Kilo 补充了"审查带宽"作为具体停止规则
- [元认知](<../../教育/学习方法/元认知.md>) — 对自身审查能力的判断是元认知在 AI 协作中的体现
- [AI 学习路由](<../教育AI/AI 学习路由.md>) — 同为 AI 系统设计原则，方向不同（教育路由 vs 工程分解）

## 延伸阅读

- [Kilo 官网](https://kilo.ai)
- [Kilo Blog: What We Learned from 3 Million Downloads](https://blog.kilo.ai/p/what-we-learned-from-3-million-downloads)
- [Reviewability as the Bottleneck](https://tessl.io/blog/the-model-is-solved-now-comes-the-hard-part-reviewability-as-the-bottleneck)
