# 🌐 VPN Gateway: 高可用家庭科学上网实战方案

## 1. 核心目标
建立一个基于 **阿派 (Local Gateway) + 老谷 (Google VM) + Cloudflare (CDN)** 的三位一体科学上网链路。
*   **低成本**：利用 Google Cloud 免费层 (`e2-micro`) 及 Cloudflare 免费 CDN。
*   **高隐蔽**：通过 Cloudflare CDN 隐藏老谷真实 IP，使用 WebSocket+TLS 伪装流量。
*   **省钱策略**：利用 Cloudflare Peering 规避 GCP 直连中国大陆的高昂流量费。
*   **全家无感**：阿派作为热点/网关，实现家庭设备免客户端接入。

## 2. 流量拓扑与“双引擎”现状

### 2.1 链路 A：Cloudflare CDN 转发流 (老谷 + CF Proxy) —— 核心稳定链路
*   **路径**：`[终端设备]` -> `[🐾 阿派]` -> `[WS+TLS]` -> `[🟠 Cloudflare 边缘]` -> `[👴 老谷 (Port 8443)]`
*   **协议细节**：VLESS / VMess + WebSocket (传输层) + TLS。
*   **当前配置**：
    *   **老谷端**：监听 `8443` 端口（Cloudflare 官方支持的回源端口）。
    *   **资源加固**：已开启 **2GB Swap** 解决 `e2-micro` 内存溢出问题。
    *   **安全策略**：关闭非必要入站端口，由 CF 节点分担 DDoS 风险。
*   **备注**：*原计划的 Cloudflare Tunnel 因信用卡绑定限制已废弃，当前方案采用 CDN 转发。*

### 2.2 链路 B：UDP 暴力流 (传家宝 VPS) —— 速度担当链路
*   **路径**：`[终端设备]` -> `[🐾 阿派]` -> `[Hysteria2 (UDP)]` -> `[💾 传家宝 VPS]`
*   **协议选型**：Hysteria2 (Hy2)。
*   **核心优势**：基于 QUIC/UDP，在丢包严重的线路上依然能强制跑满带宽，适合 4K 视频。

## 3. 本地网关 (阿派) 配置 🐾
*   **硬件**：树莓派 (arm64)，V2Ray v5.37.0。
*   **管理**：运行 `v2rayA` 进行图形化分流。
*   **加固**：控制面板 API 仅监听局域网地址 `192.168.0.105:2017`，禁止公网访问。
*   **资源**：同样配置了 **2GB Swap** 确保网关稳定性。

## 4. 维护与监控
*   **健康检查**：通过阿派定期探测老谷 `8443` 端口存活及代理穿透性。
*   **仓库同步**：本地工作目录 `/home/zrlgs/vpn-gatewat` 已与 GitHub 同步。
*   **任务追踪**：所有操作历史记录在 `memory/daily-task-log.md`。

---
*Last Updated: 2026-03-06 by 阿派 (Apai)*
