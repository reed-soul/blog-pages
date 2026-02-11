---
title: "WebMCP：让浏览器 Agent 直接调用网站工具"
date: 2026-02-11T18:00:00+08:00
draft: false
description: "WebMCP 把 MCP 协议带到前端，让 AI Agent 能直接调用网站功能，无需单独的服务器。"
isStarred: false
tags: ["ai", "web", "agent", "mcp", "browser"]
categories: ["tech-notes"]
---

最近看到一个有趣的新技术：**WebMCP**（Web Model Context Protocol）。

它做的事情听起来很简单：**让 AI Agent 能够直接调用网站的功能**。

## 背景问题

现在浏览器 Agent 越来越多了，比如浏览器里内置的 AI 助手、各种浏览器扩展。这些 Agent 帮我们做很多事情：总结网页内容、填写表单、搜索信息……

但它们有个共同的问题：**只能通过"点击"来操作网页**。

举个例子，你想让 Agent 帮你找一件 200 元以下的红色连衣裙：

- Agent 需要识别页面上的搜索框
- 在输入框中输入关键词
- 点击搜索按钮
- 在结果页面中筛选价格和颜色
- 逐个点击查看详情

整个过程就像一个"瞎子"，只能通过点击和屏幕识别来操作。这种方式的问题很明显：

1. **效率低**：需要多次点击和等待页面加载
2. **不稳定**：页面结构一变，Agent 可能找不到按钮
3. **难维护**：每个网站都要专门训练 Agent 识别页面元素

## MCP 的出现

为了解决这个问题，业界提出了 **MCP 协议**（Model Context Protocol）。

MCP 的想法是：**让网站把自己的功能暴露为结构化的工具**，Agent 可以直接调用这些工具。

比如，一个电商网站可以暴露这样的工具：

```json
{
  "name": "search_products",
  "description": "搜索商品",
  "parameters": {
    "keywords": "搜索关键词",
    "maxPrice": "最高价格",
    "color": "颜色"
  }
}
```

Agent 收到用户的请求后，直接调用这个工具，网站返回搜索结果，整个过程不需要点击任何按钮。

这个想法很好，但 **MCP 最初是为后端设计的**。网站需要搭建一个单独的 MCP 服务器，这增加了开发和维护成本。

## WebMCP：MCP 来到前端

**WebMCP** 把 MCP 的理念带到了前端。

它的核心思想很简单：**用纯 JavaScript（甚至 HTML）就能把网站功能暴露为结构化工具**，无需单独的服务器。

### 工作原理

网站只需要在页面上定义一个特殊的 JavaScript 对象：

```javascript
window.webMCP = {
  tools: {
    searchProducts: {
      name: "搜索商品",
      description: "根据关键词、价格、颜色等条件搜索商品",
      parameters: {
        type: "object",
        properties: {
          keywords: { type: "string", description: "搜索关键词" },
          maxPrice: { type: "number", description: "最高价格" },
          color: { type: "string", description: "颜色" }
        }
      },
      handler: async (params) => {
        // 调用网站的搜索 API
        const results = await window.shopAPI.search(params);
        return results;
      }
    }
  }
};
```

当浏览器 Agent 访问这个页面时，它会：

1. **发现工具**：自动识别 `window.webMCP` 对象
2. **理解功能**：从工具的描述中理解它做什么
3. **调用工具**：直接执行 `handler` 函数
4. **返回结果**：把结果返回给用户

整个过程用户可以看到发生了什么，并且可以随时中断。

### 实际例子

还是刚才的例子：找一件 200 元以下的红色连衣裙。

**传统方式**（点击）：
1. Agent 识别搜索框 → 输入关键词 → 点击搜索
2. 等待页面加载 → 识别筛选器 → 设置价格范围
3. 等待结果更新 → 识别筛选器 → 选择颜色
4. 等待结果更新 → 逐个点击查看详情

**WebMCP 方式**（直接调用）：
```
Agent: 帮我找一件 200 元以下的红色连衣裙
→ Agent 调用 searchProducts({ keywords: "连衣裙", maxPrice: 200, color: "红色" })
→ 网站返回搜索结果
```

对比一下：

| 维度 | 传统方式 | WebMCP |
|------|---------|---------|
| 操作次数 | 5-10 次点击 | 1 次调用 |
| 等待时间 | 多次页面加载 | 1 次请求 |
| 稳定性 | 依赖页面结构 | 不依赖 |
| 开发成本 | 每个网站单独训练 | 一次定义工具 |

## 三方收益

### 对开发者

**最大的好处是"不用搭服务器"**。

传统的 MCP 需要一个后端服务器来暴露工具，而 WebMCP 直接在前端定义：

- **复用现有代码**：网站已经有搜索 API，直接用就行
- **无需学习新框架**：就是普通的 JavaScript
- **部署零成本**：工具定义在前端，和网站一起部署

### 对用户

用户体验会明显提升：

- **更快**：直接调用工具，不需要多次页面加载
- **更可靠**：工具调用是确定的，不会因为页面结构变化而失效
- **可控制**：用户可以看到 Agent 在调用什么工具，随时可以中断

### 对网站

网站开发者对 Agent 的交互有了更多控制权：

- **保持品牌体验**：Agent 调用的是网站自己的工具，不会破坏 UI
- **控制认证流程**：敏感操作仍然走网站自己的登录
- **定义交互方式**：网站决定 Agent 可以做什么，不能做什么

## Human-in-the-loop 设计

WebMCP 的一个核心理念是：**人始终在控制中**。

Agent 调用工具时：

1. 用户可以看到 Agent 调用了什么工具
2. 用户可以看到调用结果
3. 用户可以随时中断 Agent 的操作

这不是"盲目的自动化"，而是"增强的交互"。

## 现状

WebMCP 目前是一个 **W3C 社区小组草案**，属于 Web Machine Learning CG。

好消息是：**Chrome 146 已经提供了早期预览**，开发者可以通过 `navigator.modelContext` API 来原型开发。

GitHub 仓库：https://github.com/webmachinelearning/webmcp

## 总结

WebMCP 的核心思想可以概括为一句话：

> **让网站的功能能够被 AI Agent 直接调用，而不是通过点击屏幕来操作。**

这个想法看似简单，但解决了一个根本问题：**让浏览器 Agent 从"操作员"变成了"工具调用者"**。

对开发者来说，这意味着不用搭服务器，复用现有代码。
对用户来说，这意味着更快、更可靠的交互。
对网站来说，这意味着对 Agent 交互的控制权。

这是一个值得关注的趋势，可能会改变我们和浏览器 Agent 交互的方式。

## 参考

- WebMCP GitHub: https://github.com/webmachinelearning/webmcp
- WebMCP 介绍推文: https://x.com/i/status/2021289124997554534
- MCP 协议: https://modelcontextprotocol.io/
