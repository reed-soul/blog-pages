---
title: "OpenClaw + Agent Swarm：一人开发团队的完整搭建指南"
date: 2026-02-24T20:00:00+08:00
draft: false
tags: ["OpenClaw", "AI Agent", "Claude Code", "Codex", "自动化", "翻译"]
categories: ["AI工具"]
description: "如何用 OpenClaw 作为编排层，管理多个 AI agents 实现一人开发团队。从客户需求到生产部署的完整工作流。"
translation: true
translationSource: "https://x.com/elvissun/status/2025920521871716562"
translationAuthor: "Elvis (@elvissun)"
---

> **📌 翻译说明**：本文翻译自 Elvis 的推文 [OpenClaw + Codex/ClaudeCode Agent Swarm: The One-Person Dev Team](https://x.com/elvissun/status/2025920521871716562)，作者分享了如何用 OpenClaw 构建一个 AI agent 编排系统，实现真正的一人开发团队。

<!--more-->

我现在不再直接使用 Codex 或 Claude Code 了。

我用 OpenClaw 作为编排层。我的 orchestrator，Zoe，负责生成 agents、编写 prompts、为每个任务选择合适的模型、监控进度，并在 PR 准备好合并时通过 Telegram 通知我。

**过去 4 周的成果：**

- **一天 94 次提交**。这是我最高产的一天——当天我还有 3 个客户电话，一次都没打开编辑器。平均每天约 50 次提交。
- **30 分钟内 7 个 PR**。从想法到生产部署极快，因为编码和验证基本都自动化了。
- **Commits → MRR**：我用这个系统构建一个真实的 B2B SaaS——结合创始人主导的销售，大多数功能请求当天就能交付。速度能把潜在客户转化为付费用户。

> 📊 **图表**：原文包含一张 git 提交历史对比图，展示了使用 OpenClaw 前后的提交频率变化——1月之前只有 CC/codex，1月之后 OpenClaw 编排了 CC/codex agents。[查看原图](https://x.com/elvissun/status/2025920521871716562)

我的 git 历史看起来就像我刚雇佣了一个开发团队。实际上只是我——从直接管理 Claude Code，变成了管理一个 OpenClaw agent，而这个 agent 管理着一群其他的 Claude Code 和 Codex agents。

**成功率**：系统几乎能一次性完成所有中小型任务，无需任何干预。

**成本**：Claude 约 $100/月，Codex 约 $90/月，但你可以从 $20 开始。

## 为什么这个方案比直接用 Codex 或 Claude Code 更好

> Codex 和 Claude Code 对你的业务几乎没有任何上下文。

它们只看到代码。它们看不到你业务的全貌。

OpenClaw 改变了这个等式。它作为你与所有 agents 之间的编排层——它在我的 Obsidian vault 中保存了所有业务上下文（客户数据、会议笔记、过去的决策、什么有效、什么失败），并将历史上下文转化为每个编码 agent 的精准 prompts。Agents 专注于代码。Orchestrator 保持在高层策略层面。

## 高层系统架构

> 📊 **架构图**：原文包含一张系统架构图，展示了 OpenClaw 如何作为编排层管理多个 coding agents。[查看原图](https://x.com/elvissun/status/2025920521871716562)

上周 Stripe 写了他们的后台 agent 系统 "Minions"——由中央编排层支持的并行编码 agents。我不小心构建了同样的东西，但它运行在我的 Mac mini 上。

在我告诉你如何设置之前，你应该知道**为什么**你需要一个 agent orchestrator。

## 为什么一个 AI 无法同时做两件事

**上下文窗口是零和博弈**。你必须选择放什么进去。

- 填满代码 → 没有空间放业务上下文
- 填满客户历史 → 没有空间放代码库

这就是为什么两层系统有效：每个 AI 都加载了它精确需要的内容。

> 📊 **上下文对比图**：原文包含一张 OpenClaw vs Codex 的上下文对比图，展示了两者加载的不同类型信息。[查看原图](https://x.com/elvissun/status/2025920521871716562)

**通过上下文实现专业化，而不是通过不同的模型。**

## 完整的 8 步工作流

让我用一个上周的真实例子来说明。

### Step 1: 客户请求 → 与 Zoe 确定范围

我与一个代理商客户开了个电话会议。他们想复用团队中已经设置好的配置。

会议结束后，我与 Zoe 讨论了这个请求。因为我所有的会议笔记都会自动同步到我的 Obsidian vault，我不需要解释任何背景。我们一起确定了功能范围——设计了一个模板系统，让他们可以保存和编辑现有配置。

**然后 Zoe 做了三件事：**

1. **充值积分**——立即解除客户限制（她有 admin API 访问权限）
2. **从生产数据库拉取客户配置**——她有只读生产数据库访问权限（我的 codex agents 永远不会有这个），用于检索他们的现有设置，这些会包含在 prompt 中
3. **生成一个 Codex agent**——带有包含所有上下文的详细 prompt

### Step 2: 生成 Agent

**每个 agent 都有自己的 worktree（隔离分支）和 tmux session：**

Agent 在 tmux session 中运行，通过脚本记录完整的终端日志。

**我们如何启动 agents：**

我以前用 `codex exec` 或 `claude -p`，但最近切换到了 tmux：

**tmux 好得多，因为中途重定向很强大**。Agent 走错方向了？别杀掉它：

任务被跟踪在 `.clawdbot/active-tasks.json` 中：

完成时，它会更新 PR 号和检查状态。（更多内容在 step 5）

### Step 3: 循环监控

一个 cron job 每 10 分钟运行一次来照看所有 agents。这基本上是一个改进版的 Ralph Loop，后面会详细说。

但它不直接轮询 agents——那样太昂贵了。相反，它运行一个脚本，读取 JSON 注册表并检查：

**这个脚本是 100% 确定性的，极其节省 token：**

- 检查 tmux sessions 是否存活
- 检查跟踪分支上是否有开放的 PRs
- 通过 gh cli 检查 CI 状态
- 如果 CI 失败或有关键审查反馈，自动重启失败的 agents（最多 3 次尝试）
- 只在需要人工关注时发出警报

我不盯着终端。系统会告诉我什么时候该看了。

### Step 4: Agent 创建 PR

Agent 通过 `gh pr create --fill` 提交、推送并打开 PR。**此时我不会收到通知——仅有一个 PR 还不算完成。**

**完成定义（非常重要，你的 agent 需要知道这个）：**

- PR 已创建
- 分支已同步到 main（无合并冲突）
- CI 通过（lint、types、unit tests、E2E）
- Codex review 通过
- Claude Code review 通过
- Gemini review 通过
- 包含截图（如果是 UI 变更）

### Step 5: 自动化代码审查

**每个 PR 都会被三个 AI 模型审查。它们会发现不同的问题：**

1. **Codex Reviewer**——在边缘情况方面表现出色。做最彻底的审查。能发现逻辑错误、缺失的错误处理、竞态条件。误报率很低。

2. **Gemini Code Assist Reviewer**——免费且极其有用。能发现其他 agents 遗漏的安全问题、可扩展性问题。并建议具体的修复方案。安装它是理所当然的。

3. **Claude Code Reviewer**——基本没用——倾向于过度谨慎。很多"考虑添加..."的建议通常是过度工程。除非标记为关键，否则我跳过所有内容。它很少自己发现关键问题，但会验证其他审查者标记的内容。

所有三个都会直接在 PR 上发布评论。

### Step 6: 自动化测试

**我们的 CI 流水线运行大量自动化测试：**

- Lint 和 TypeScript 检查
- 单元测试
- E2E 测试
- Playwright 测试（针对预览环境，与生产环境相同）

我上周添加了一条新规则：如果 PR 更改了任何 UI，必须在 PR 描述中包含截图。否则 CI 失败。这大大缩短了审查时间——我可以准确看到什么变了，而不用点击预览。

### Step 7: 人工审查

现在我收到 Telegram 通知："PR #341 ready for review."

**到这个时候：**

- CI 通过
- 三个 AI reviewers 批准了代码
- 截图显示了 UI 变化
- 所有边缘情况都记录在审查评论中

我的审查需要 5-10 分钟。很多 PR 我不看代码就直接合并——截图就能告诉我需要知道的一切。

### Step 8: 合并

PR 合并。一个每日 cron job 清理孤立的 worktrees 和任务注册 JSON。

## Ralph Loop V2

这本质上是 Ralph Loop，但更好。

Ralph Loop 从记忆中拉取上下文，生成输出，评估结果，保存学习。但大多数实现每个循环运行相同的 prompt。提炼的学习改进未来的检索，但 prompt 本身保持静态。

**我们的系统不同。** 当 agent 失败时，Zoe 不会只用相同的 prompt 重启它。她会用完整的业务上下文查看失败，并找出如何解除阻塞：

- Agent 上下文用完了？"只关注这三个文件。"
- Agent 走错方向了？"停。客户想要的是 X，不是 Y。这是他们在会议上说的。"
- Agent 需要澄清？"这是客户的邮件和他们公司的业务。"

Zoe 照看 agents 直到完成。她有 agents 没有的上下文——客户历史、会议笔记、我们之前尝试过什么、为什么失败。她用这些上下文在每次重试时写更好的 prompts。

**但她也不会等我分配任务。她会主动找活干：**

- **早上：** 扫描 Sentry → 发现 4 个新错误 → 生成 4 个 agents 调查和修复
- **会议后：** 扫描会议笔记 → 标记客户提到的 3 个功能请求 → 生成 3 个 Codex agents
- **晚上：** 扫描 git log → 生成 Claude Code 更新 changelog 和客户文档

我在客户电话后去散步。回来看到 Telegram："7 个 PR 准备审查。3 个功能，4 个 bug 修复。"

当 agents 成功时，模式会被记录。"这种 prompt 结构对计费功能有效。""Codex 需要预先提供类型定义。""总是包含测试文件路径。"

奖励信号是：CI 通过、所有三个代码审查通过、人工合并。任何失败都会触发循环。随着时间推移，Zoe 写出更好的 prompts，因为她记得什么被发布了。

## 选择合适的 Agent

并非所有编码 agents 都是平等的。快速参考：

- **Codex** 是我的主力。后端逻辑、复杂 bugs、多文件重构，任何需要跨代码库推理的任务。它较慢但彻底。我用它处理 90% 的任务。

- **Claude Code** 更快，更擅长前端工作。它的权限问题也更少，所以很适合 git 操作。（我以前更多用它处理日常工作，但 Codex 5.3 现在简单更好更快）

- **Gemini** 有不同的超能力——设计感。对于漂亮的 UI，我会先让 Gemini 生成一个 HTML/CSS 规范，然后交给 Claude Code 在我们的组件系统中实现。Gemini 设计，Claude 构建。

Zoe 为每个任务选择合适的 agent 并在它们之间路由输出。计费系统 bug 交给 Codex。按钮样式修复交给 Claude Code。新仪表板设计从 Gemini 开始。

## 如何设置

**把整篇文章复制到 OpenClaw 并告诉它："为我的代码库实现这个 agent swarm 设置。"**

它会读取架构、创建脚本、设置目录结构、配置 cron 监控。10 分钟搞定。

没有课程卖给你。

## 没人预料到的瓶颈

**我现在碰到的天花板是：RAM。**

每个 agent 需要自己的 worktree。每个 worktree 需要自己的 `node_modules`。每个 agent 运行构建、类型检查、测试。五个 agents 同时运行意味着五个并行 TypeScript 编译器、五个测试运行器、五组依赖加载到内存中。

我的 16GB Mac Mini 在 4-5 个 agents 时就会触顶——开始交换内存——而且需要幸运的是它们不会同时尝试构建。

所以我买了一台 128GB RAM 的 Mac Studio M4 max（$3,500）来驱动这个系统。它三月底到货，我会分享是否值得。

## 下一步：一人百万美元公司

从 2026 年开始，我们会看到大量一人百万美元公司。对于那些理解如何构建递归自我改进 agents 的人来说，杠杆是巨大的。

**它看起来是这样的：** 一个 AI orchestrator 作为你自己的延伸（就像 Zoe 对我一样），将工作委托给处理不同业务功能的专门 agents。工程。客户支持。运营。营销。每个 agent 专注于它擅长的事情。你保持极度专注和完全控制。

下一代创业者不会雇佣 10 人的团队来做一个人用正确系统能做的事。他们会这样构建——保持小规模、快速移动、每天发布。

现在有太多 AI 生成的垃圾了。太多围绕 agents 和"任务控制"的炒作，却没有构建任何真正有用的东西。花哨的演示，没有实际收益。

**我正在尝试做相反的事情：少炒作，更多关于构建真实业务的文档。真实客户、真实收入、真实发布到生产的提交，以及真实的损失。**

我在构建什么？**Agentic PR**——一个一人公司，挑战企业 PR 老牌公司。帮助初创公司获得媒体报道的 agents，无需 $10k/月的 retainers。

如果你想看看我能走多远，欢迎关注。

---

> 💡 **译者注**：这篇文章展示了 AI agent 编排的实际应用场景。OpenClaw 作为编排层的思路很有价值——它不是替代 Codex/Claude Code，而是让它们更好地协同工作。对于独立开发者来说，这种"一人团队"的模式值得借鉴。
