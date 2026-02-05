---
title: "antigravity-proxy：不启用 TUN 的进程级代理方案"
date: 2026-02-05T10:00:00+08:00
draft: false
description: "基于 Windows DLL 注入的透明代理：只代理目标进程，避免全局 TUN。"
isStarred: false
tags: ["network", "proxy", "antigravity", "clash"]
categories: ["tech-notes"]
---

我在大陆用 Google Antigravity，需要代理。以前一直靠 Clash 的 TUN 模式顶住，但负载明显上去，而且和公司 VPN 也容易打架。后来我想换一种更“轻”的方案：**只让 Antigravity 走代理，不碰系统路由**。于是找到了 antigravity-proxy。

仓库地址：<https://github.com/yuaotian/antigravity-proxy>

下面是我基于 README 做的专业整理，重点放在“它能做什么、怎么做、边界在哪里”。

## 项目定位

antigravity-proxy 是一个**面向 Antigravity 的 Windows 代理注入组件（DLL）**。核心目标很清晰：  
**在不启用 TUN 的情况下，让 Antigravity 进程稳定走 SOCKS5/HTTP 代理。**

它不接管全局流量，只代理指定进程，因此更适合与公司 VPN 共存。

## 核心能力（整理自 README）

- **平台与架构**：Windows x86 / x64。
- **免 TUN**：不走系统级虚拟网卡，避免全局路由接管。
- **进程级代理**：仅代理指定进程（默认聚焦 Antigravity）。
- **透明代理**：目标程序无感知，配置到位即可生效。
- **协议支持**：SOCKS5 / HTTP，README 建议优先 SOCKS5。
- **配置驱动**：通过 `config.json` 控制代理地址、规则、模式。
- **日志与排错**：内置日志路径与排查步骤，适合快速定位问题。
- **高级能力**（可选）：路由规则、UDP/QUIC 处理、IPv6 行为控制、子进程注入等。

## 工作原理（高层视角）

它通过 `version.dll` 的加载机制进入目标进程，Hook Windows 的网络相关 API，把连接转发到你指定的代理上。  
你可以把它理解为“**只对目标进程生效的透明代理层**”。

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': {'primaryColor': '#1f2937', 'primaryTextColor': '#f9fafb', 'lineColor': '#93c5fd', 'secondaryColor': '#111827'}}}%%
flowchart LR
  A[Antigravity 进程] --> B[version.dll 注入]
  B --> C[Hook Winsock 连接]
  C --> D[SOCKS5/HTTP 代理]
  D --> E[外网]
```

## 快速上手（概念版）

1. 准备本机可用的 SOCKS5/HTTP 代理（Clash/Mihomo 等）。
2. 从 Release 获取 `version.dll` 与 `config.json`。
3. 将这两个文件放到 Antigravity 主程序目录（与 `Antigravity.exe` 同级）。
4. 启动 Antigravity，观察日志确认代理是否生效。

注意事项（README 有明确提示）：
- **x86/x64 要匹配**：DLL 位数需与目标程序一致。
- **运行库缺失会报错**：例如 0xC0000142，需要安装 VC++ 运行库。

## 适用边界

- 这是 **Windows 方案**，对 WSL 内部流量不生效（README 有专门说明）。
- 如果你的场景是 WSL，仓库里给了替代思路，可参考其 WSL 章节。

## 我的判断

对我来说，这个方案的价值很明确：

1. **范围可控**：只代理目标进程，避免全局污染。
2. **与 VPN 共存**：不碰系统路由，冲突成本低。
3. **可回退**：删掉 DLL 和配置，系统即恢复原状。

当然，它也意味着你需要对“注入 + Hook”这类工具保持足够谨慎，合规和安全永远是第一位。

## 结语

如果你的目标是“让 Antigravity 能用，但不想开全局 TUN”，  
antigravity-proxy 是一个非常值得认真看的方案。

---

**参考**  
- antigravity-proxy 仓库：<https://github.com/yuaotian/antigravity-proxy>
