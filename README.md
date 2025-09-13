# Unbound 一键安装表述文件

本文件用于说明 **Unbound 一键安装脚本** 的设计目标、功能范围、安装环境、文件结构以及运维管理方式，便于理解与后续维护。无具体执行脚本，仅为表述性文档。

---

## 📌 设计目标

* 提供快速部署 Unbound 的一键化流程。
* 兼容主流 Linux 发行版（Ubuntu/Debian、CentOS/RHEL/Alma/Rocky）。
* 生成安全、优化的递归解析器配置，支持 DNSSEC 与 QNAME 最小化。
* 自动更新根提示文件，减少人工维护。
* 提供便捷的管理指令（安装、卸载、更新、服务控制、日志）。

---

## ✅ 功能说明

1. **系统适配**
   自动检测操作系统类型与包管理器（apt/dnf/yum）。

2. **安装与卸载**

   * 安装 `unbound` 及依赖工具（如 curl、证书包）。
   * 卸载时保留配置文件目录，避免数据丢失。

3. **配置管理**

   * 自动生成主配置文件 `/etc/unbound/unbound.conf`。
   * 启用 DNSSEC、最小化查询、缓存优化、最小响应、预取缓存等安全性能特性。
   * 支持 IPv4/IPv6，默认监听 `53/udp` 与 `53/tcp`。
   * 默认仅允许本机与内网访问，避免对公网直接开放。

4. **根提示更新**
   自动下载并更新最新的 `root.hints` 文件。

5. **安全与防火墙**
   如检测到 `ufw` 或 `firewalld`，会自动开放 53 端口。

6. **服务管理**

   * 使用 `systemd` 管理服务。
   * 提供启停、重启、状态查询、日志查看等便捷操作。

7. **可选功能**

   * 转发到 DoT（DNS over TLS）或 DoH（DNS over HTTPS）上游。
   * 支持自定义内网域名解析与广告屏蔽。

---

## 📁 文件结构

* `/etc/unbound/unbound.conf` —— 主配置文件。
* `/etc/unbound/unbound.conf.d/` —— 片段配置目录。
* `/etc/unbound/root.hints` —— 根提示文件。
* `/var/lib/unbound/root.key` —— DNSSEC 信任锚。
* `unbound` —— systemd 服务名。

---

## 🔍 使用方式

提供以下操作命令（逻辑定义，不含具体实现）：
一键安装脚本如下

bash <(curl -fsSL https://raw.githubusercontent.com/wuxingzhidi/ubound-dns-one-click-installation-script/main/dns-Unbound)

bash <(curl -fsSL https://raw.githubusercontent.com/wuxingzhidi/ubound-dns-one-click-installation-script/main/kill-53)
* `install` —— 安装并初始化 Unbound。
* `uninstall` —— 卸载（保留配置文件）。
* `update-hints` —— 更新根提示文件并重启服务。
* `status` —— 查看服务状态。
* `restart` —— 重启服务。
* `start` —— 启动服务。
* `stop` —— 停止服务。
* `logs` —— 查看服务日志。

---

## 🔐 安全与性能建议

* 保持默认仅内网可访问，若需对外提供服务，请务必加上 QPS 限速和防护策略。
* 根据服务器 CPU 自动调整线程数（推荐 = CPU 核心数，1–8 之间）。
* 若启用 DoT/DoH 上游，建议指定可信服务器并启用证书验证。

---

## 🆘 常见问题

* **端口冲突**：若 53 端口被占用，需停止 `systemd-resolved` 或 `dnsmasq` 等服务。
* **IPv6 支持**：若环境不支持 IPv6，可在配置中禁用 `do-ip6`。
* **自定义域名**：可在配置文件中加入 `local-zone` 与 `local-data` 实现内网解析。

---
