---
title: "免 TUN 的 Antigravity：一次更轻的代理尝试"
date: 2026-02-05T10:00:00+08:00
draft: false
description: "记录在大陆使用 Antigravity 时，如何避开 Clash TUN 带来的负载与 VPN 冲突。"
isStarred: false
tags: ["network", "proxy", "antigravity", "clash"]
categories: ["tech-notes"]
---

这篇文章只解决一个问题：在中国大陆使用 Google 的 Antigravity 时，能不能不启用 Clash 的 TUN 模式？

我的答案是：可以尝试把“系统级代理”改成“进程级代理”，只让目标程序走代理，不动系统路由。

## 背景

- **场景**：使用 Antigravity 需要代理能力。
- **现实限制**：Clash 的 TUN 模式会接管系统流量，网络负载更高。
- **冲突点**：公司 VPN 同时在改路由，容易相互干扰。

## 为什么我想避开 TUN

TUN 本质上是虚拟网卡。它的优点是“全局可控”，但副作用也明显：

- 代理范围过大：不需要代理的流量也被接管。
- 负载增加：路由、分流和加密都要额外处理。
- 与公司 VPN 冲突：两套“全局路由”很容易打架。

我更想要的是“**只代理 Antigravity**”，而不是“代理所有流量”。

## 发现 antigravity-proxy

我后来发现了这个仓库：<https://github.com/yuaotian/antigravity-proxy>。

根据仓库简介，它是一个 **Transparent proxy injector for Antigravity**，用于在 **Windows** 上 **免 TUN 强制代理**，支持 **SOCKS5/HTTP**，并提到 **DLL 注入与进程流量劫持**。

这正好命中我的需求：**进程级、免 TUN、目标明确**。

## 我的判断标准

我给自己定了三个标准，确保方案更“轻”也更稳：

1. **不改系统路由**：避免干扰 VPN。
2. **只代理目标进程**：减少无关流量。
3. **能回退**：发现问题可以快速恢复原配置。

antigravity-proxy 至少在“思路”上符合这些点。

## 我的使用记录（简化版）

为了避免踩坑，我按下面的顺序做了小范围验证：

1. 先读仓库说明，确认支持的系统与代理类型。
2. 准备一个可用的 SOCKS5/HTTP 代理。
3. 只对 Antigravity 进程做代理，不改系统全局设置。
4. 观察 VPN 与网络负载是否有异常。

这一步我只做了“功能可用性”和“冲突风险”的判断，没有做更深入的性能测试。

## 风险与提醒

- **合规与安全**：涉及代理、注入等行为前，请确认合规与公司政策。
- **安全性**：任何注入类工具都要格外小心，优先看源码与社区反馈。
- **可控性**：别追求“全局代理”，能控制范围才是关键。

## 结语

我的体会很简单：  
能不用 TUN，就别用 TUN；能不动系统路由，就尽量不动。

**把代理范围缩小到“目标进程”，往往是对稳定性最友好的选择。**

---

**参考**  
- antigravity-proxy 仓库：<https://github.com/yuaotian/antigravity-proxy>
