---
title: "Cursor 2026：13 个提升开发效率的实用技巧"
date: 2026-02-20T10:00:00+08:00
draft: false
tags: ["Cursor", "AI", "开发工具", "效率提升", "翻译"]
categories: ["工具"]
description: "从 MCP 集成到 Slack 机器人，这些是 Cursor 高级用户每天都在用的核心技巧。"
translation: true
translationSource: "https://youtu.be/7tpEsI93PfE"
translationAuthor: "YouTube"
---

作为一名每天使用 Cursor 超过 12 小时的开发者，我整理了 13 个能显著提升开发效率的实用技巧。这些都是我在日常工作流中反复验证过的方法。

> **📌 翻译说明**：本文翻译自 [13 Cursor Tips Every Developer NEEDS to Know](https://youtu.be/7tpEsI93PfE)，原作者在视频中分享了这些 Cursor 高级技巧。

<!--more-->

## 1. MCP（Model Context Protocol）集成

MCP 让你的 AI 助手能够访问外部数据源，获取实时信息。

**推荐使用：**
- **XRMCP** - 让 AI 搜索网页，获取最新的技术文档
- **Xcode Build MCP** - 如果你在开发 iOS 应用，这个工具可以直接在 Cursor 中构建项目

当你的 AI 助手需要查询最新的 API 文档或技术规范时，这些 MCP 工具能确保它不会使用过时的信息。

## 2. 自定义斜杠命令

对于重复性的工作流，创建斜杠命令是最好的选择。

**我的常用命令：**
- `/create-pr` - 自动创建 PR
- `/commit-chunks` - 分块提交代码
- `/pr-comments-autofix` - 拉取 PR 审查评论，自动判断是否需要修复，然后推送修复

**原则**：任何你每天都会做的事情，都应该变成斜杠命令。

## 3. Ask Mode（询问模式）

在开始一个新任务时，我总是先用 Ask Mode 来获取上下文。

Ask Mode 的特点：
- AI 可以回答关于代码库的问题
- **不会对代码做任何修改**
- 帮助你快速了解不熟悉的模块

**工作流**：新任务 → Ask Mode 获取上下文 → 制定更好的计划 → 执行

## 4. Plan Mode（规划模式）

对于任何非简单任务，都应该使用 Plan Mode。

Plan Mode 的价值：
- **提示词优化** - AI 会帮你完善模糊的需求
- **信息澄清** - AI 会主动询问缺失的信息
- **架构可视化** - 生成清晰的架构图和变更计划

一个简单的 "add minesweeper game" 需求，经过 Plan Mode 会变成包含文件列表、实现步骤、架构设计的详细方案。

## 5. Fork Chat（对话分叉）

当你完成了计划的实施，想问后续问题，但又不想污染主对话上下文时：

点击对话下方的三个点 → **Fork Chat**

这样可以在副本中提问，保持主线对话的清晰。

## 6. Debug Mode（调试模式）

这是 Cursor 最强大的功能之一。

**工作原理**：
1. AI 提出多个假设（比如 5 个可能的原因）
2. 添加调试日志（instrumentation）来验证假设
3. 让用户重现问题
4. AI 分析日志，定位根本原因
5. 自动修复或添加更多调试信息

**示例提示词**：
```
Debug why clicking a tile in nonogram isn't doing anything
```

## 7. 模型切换的正确姿势

**常见错误**：在对话中途切换模型。

为什么不好：
- 切换模型会破坏上下文缓存（Caching）
- 整个对话历史会被重新发送给新模型
- 消耗大量额外的 tokens

**正确做法**：开始一个新的对话。

## 8. Agent Review Mode（代码审查模式）

两种审查模式：
- **Quick** - 快速扫描，找出明显的 bug，适合提交前检查
- **Deep** - 深度审查，类似 Bugbot，适合 PR 最终审查

**我的习惯**：
- 提交 PR 前 → Quick Review
- PR 准备合并前 → Deep Review

## 9. 显示用量统计

很多人不知道这个功能藏在设置里。

**开启方法**：
1. 打开 Cursor Settings
2. 进入 Agents 标签页
3. 滚动到底部
4. 将 "Usage Summary" 设为 "Always"

这样每次 AI 回复后，你都能看到消耗的 token 数量。

## 10. 理解新的计费界面

Cursor Ultra 计划现在将用量分为：
- **Auto & Composer** - 独立配额，用完为止，不影响 API 用量
- **API Usage** - 包括 Codex、Opus、Gemini 等模型

目前 Composer 有 6 倍的临时加成，重度使用后可能只消耗 2% 的配额。

## 11. Bugbot - AI 代码审查

[$40/月](https://bugbot.ai)，这是我用过的最好的 AI 代码审查工具。

**特点**：
- 自动审查每个 PR 和每次提交
- 几乎零误报（False Positive）
- **Autofix 功能** - 自动创建修复 PR

准确率高到什么程度？唯一误报的情况是涉及产品决策或业务上下文，这些确实不是 AI 能判断的。

## 12. Cursor Slack Agents

在 Slack 中直接让 AI 帮你修 bug。

**使用方法**：
1. 在 Slack 线程中 @cursor 机器人
2. 描述问题或 bug
3. AI 自动调查并创建 PR
4. 可以在 Slack 或 GitHub 中继续跟进

**适用场景**：小 bug、快速修复、需要调查的问题。

## 13. Plugin Marketplace（插件市场）

Cursor 最新的重大更新。

**插件包含**：
- Skills - 针对特定技术栈的最佳实践
- Rules - 代码规范
- Sub Agents - 专门的子代理
- Hooks - 自动化钩子
- MCPs - 数据源集成

**已支持的厂商**：Vercel、Linear、Convex、Neon、Stripe

**实际效果**：有 Stripe 插件的情况下，可以通过 Cursor Agent 从零创建一个带支付功能的应用并部署到生产环境。

## Bonus：同窗口多项目

如果你厌倦了在 5 个 Cursor 窗口之间切换：

**设置方法**：
1. 打开 User Settings (JSON)
2. 添加：`"window.nativeTabs": true`
3. 重新启动 Cursor

现在你可以在同一个窗口中打开多个项目，通过标签页切换。

**快捷键设置**：
在 Keyboard Shortcuts 中配置 "Show Previous Window" 和 "Show Next Window"，用快捷键在项目间快速切换。

---

## 总结

这 13 个技巧可以分为三类：

1. **效率工具**：斜杠命令、Fork Chat、Native Tabs
2. **质量保证**：Plan Mode、Debug Mode、Agent Review、Bugbot
3. **生态集成**：MCP、Plugin Marketplace、Slack Agents

**我的建议**：不要一次性尝试所有技巧。先从 Ask Mode 和 Plan Mode 开始，这两个会改变你使用 AI 编程的方式。

---

*参考视频：[13 Cursor Tips Every Developer NEEDS to Know](https://youtu.be/7tpEsI93PfE)*
