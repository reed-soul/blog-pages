---
title: "不用 TUN 跑 Antigravity：一次小折腾"
date: 2026-02-05T10:00:00+08:00
draft: false
description: "只给目标进程走代理，尽量不动系统路由。"
isStarred: false
tags: ["network", "proxy", "antigravity", "clash"]
categories: ["tech-notes"]
---

最近在大陆用 Google Antigravity，需要代理。我一直开着 Clash 的 TUN，能用是能用，但负载明显高，而且跟公司 VPN 经常打架。于是我想试试：**只让 Antigravity 走代理，系统路由别动**。

## 为什么想避开 TUN

TUN 是虚拟网卡，优点是全局可控，但副作用也很现实：范围太大、负载太高、容易和其他 VPN 冲突。我不需要全局，只想让目标进程可用。

## 发现 antigravity-proxy

这个仓库：<https://github.com/yuaotian/antigravity-proxy>。

它的定位很明确：在 **Windows** 上用 **进程级** 的方式给 Antigravity 强制走代理，支持 **SOCKS5/HTTP**，并提到 **DLL 注入与进程流量劫持**。一句话：**免 TUN、只管目标进程**，这正是我想要的方向。

## 我做了什么（很小范围）

我没有做复杂测试，只做了基本可用性验证：

1. 先看仓库说明，确认系统与代理类型。
2. 准备可用的 SOCKS5/HTTP 代理。
3. 只对 Antigravity 进程做代理，不改系统全局设置。
4. 看看 VPN 是否还会冲突、机器负载是否好转。

## 提醒

这类工具会涉及代理与注入，务必确认合规与安全。能控制范围就不要上全局，出了问题也更好回退。

## 结语

我的体会就一句话：  
**能只代理进程，就别动系统路由。**

---

**参考**  
- antigravity-proxy 仓库：<https://github.com/yuaotian/antigravity-proxy>
