# shrc

个人 Shadowrocket 配置。

主配置（修订版）：

```text
https://raw.githubusercontent.com/WANGXU-SU/shrc/refs/heads/main/shrc-revised.conf
```

> 注：`shrc.conf` 已废弃停用，不再维护。

## 路由与隐私策略

- 路由遵循“先阻断、再分流”：国内域名（`.cn`、`China_Domain.list`、`China.list`）、微信/QQ、大陆 B 站（已移除海外版 `BiliBiliIntl.list`）及常见银行/支付直连（`DIRECT`），`GEOIP,CN,DIRECT` 作 IP 兜底，其余一律走强制代理（`FINAL` 无 `DIRECT` 回退）。
- Claude 官方核心域名与 Anthropic 自有网段在直连规则前强制走 `🧠 Claude Code` 组，独立锁定 `🇺🇸 DMIT 节点`（正则匹配 `DMIT|dmit`，默认 `DMIT-Reality-DMIT-main`，不含 `PROXY` 和 `DIRECT`），首页切换其他节点（如日本）不影响 Claude 出口；npm/pypi/GitHub/Homebrew/Google Storage 等开发依赖走 `🚀 强制代理`。完整官方网络要求：https://code.claude.com/docs/en/network-config 。
- DNS 与流量分流绑定（`direct-dns-server` 要求 Shadowrocket `>= 2.2.92`）：代理域名使用经代理隧道的 Cloudflare/Google DoH（`1.1.1.1`/`8.8.8.8#proxy`，备用 `1.0.0.1`/`8.8.4.4#proxy`）；直连域名与节点启动解析使用阿里 IP 形式 DoH（`223.5.5.5`/`223.6.6.6`），且 `dns-direct-fallback-proxy = false` 禁止直连域名回退到代理 DNS。关闭系统 DNS 及其回退，劫持 IPv4 明文 DNS（`*:53`）并拒绝 53/853 端口。
- IPv6 禁用并增加拒绝规则；UDP 与 QUIC 按正常分流放行（避免误伤微信/QQ 传输，节点不支持 UDP 时由 `udp-policy-not-supported-behaviour = REJECT` 拒绝而非直连回退）；WebRTC 防泄露改由小火箭内置「禁用 STUN」承担。RFC 1918 私网段（`10/8`、`172.16/12`、`192.168/16`）走 `DIRECT` 以允许开 VPN 访问内网设备，`100.64.0.0/10`、`169.254.0.0/16`、`224.0.0.0/4` 直接 `REJECT`。
- 移除所有去广告重写、脚本、MITM 解密（`[MITM] enable = false`）及 `BlockHttpDNS.list` 拦截，避免干扰国内 App（如美团酒店、地图定位等）的正常解析与加载。

## 导入后必须检查

1. 确认 Shadowrocket 版本 `>= 2.2.92`；更新/重新导入上面的主配置并选中它，全局路由使用“配置”模式，首页选择日常代理节点（不能使用全局直连）。
2. 进入底部 **配置 → 点本配置右侧 `i` → 代理分组**，确认 `🇺🇸 DMIT 节点` 已选中 `DMIT-Reality-DMIT-main`，`🧠 Claude Code` 已选中 `🇺🇸 DMIT 节点`，`🚀 强制代理` 已选中 `PROXY`。
3. 打开小火箭 **设置 → UDP → 禁用 STUN**（规则层已移除 STUN/UDP 拦截，此项为当前 WebRTC 防泄露的关键开关）；并关闭 iCloud Private Relay 与会覆盖本配置 DNS/规则的其他模块，重启隧道。
4. 在设备可用时开启“包括所有网络 / Include All Networks”；它属于 App/系统 VPN 设置，不能只靠此配置文本代替。按需连接可帮助维持隧道，但不是系统防火墙。
5. 用请求日志检查 `api.anthropic.com`、`platform.claude.com`、`claude.ai` 等是否命中 `🧠 Claude Code` 并走 DMIT 出口（即使首页切到日本节点）；测试一个 CN IP 和非 CN IP 的 TCP 连接。
6. **保持隧道运行，故意使所选节点不可达**，确认 Claude、国外网站及普通 DNS 均失败，日志无 DIRECT 回退；恢复节点后再测。另测隧道被关闭/崩溃的情况，后者不能由配置文件保证阻断。
7. 在 Wi-Fi、蜂窝及切换网络后分别做 DNS/WebRTC/IPv6 泄露测试，不应出现运营商 DNS 或 WebRTC 的真实公网地址。DNS 测试看到不同公共解析器出口 IP 不一定是泄露。

## 限制与代价

- 当前配置中的部分规则（如 `hijack-dns` 与 `DST-PORT,53,REJECT` 交互、节点不可达断线行为）需在用户设备上实际抓日志验证，不能仅凭静态检查宣称“零泄露”。
- UDP 已按正常分流放开，WebRTC 防泄露高度依赖小火箭内置「禁用 STUN」及浏览器自身保护；RFC 1918 私网段默认 `DIRECT`（可访问家里路由器/NAS，不再与局域网隔离），但 `private-ip-answer = false` 仅保障按 IP 访问内网，用内网域名解析私有 IP 时需按需改为 `true` 或写入 `[Host]`。
- 直连域名使用国内阿里 DoH，代理域名使用经隧道的海外 DoH；直连 DNS 不设代理兜底，阿里 DoH 不可用时国内域名解析直接失败，不串到代理出口。
- WebRTC 的本地候选地址、任意端口的 TCP TURN 或指向 CN IP 的自定义服务无法仅靠这些网络规则彻底覆盖；浏览器支持时应禁用 WebRTC/非代理 ICE。若要保证任何应用绝不暴露真实公网 IP，就不能同时无条件允许其 CN 连接直连。
- 用网页 Stripe 给 Claude 付款时，如首页当前切在非 DMIT 节点，建议临时切回 DMIT，避免付款页与 Claude 出口不一致。
- 配置约束只对经过 Shadowrocket 的流量有效。手机 VPN 不自动代理电脑/WSL，也不保证热点客户端流量经过隧道。Claude Code 在电脑运行时，需要该电脑自己的 HTTP 代理设置或 TUN；官方支持 HTTP_PROXY/HTTPS_PROXY，不支持 SOCKS URL。自定义 ANTHROPIC_BASE_URL、第三方模型网关及 MCP 服务需另加其实际域名的强制代理规则。
- 隧道关闭、崩溃或系统绕过 VPN 时，“宁愿断网也不直连”需要 App/系统阻断或运行 Claude Code 的设备防火墙共同实现；配置文件无法单独提供系统级 kill switch。

## 说明

DNS 泄露测试的结果还会受到节点、iOS 网络环境、应用自身 DoH/HTTPDNS、IPv6 等因素影响。配置只能尽量约束 DNS 路径，不能保证所有第三方测试页面都只显示同一个 DNS 出口。
