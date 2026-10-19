# shrc

个人 Shadowrocket 配置。

主配置（修订版）：

```text
https://raw.githubusercontent.com/WANGXU-SU/shrc/refs/heads/main/shrc-revised.conf
```

> 注：`shrc.conf` 已废弃停用，不再维护。

## 路由与隐私策略

- 路由遵循“先阻断、再分流”：国内域名（`.cn`、`China_Domain.list`、`China.list`）、微信/QQ、B站及常见银行/支付直连（`DIRECT`），`GEOIP,CN,DIRECT` 作 IP 兜底，其余一律走强制代理（`FINAL` 无 `DIRECT` 回退）。
- Claude 官方核心域名、`claude.list` 远程规则集，以及安装、更新、插件常用的 GitHub/npm/pypi/Google Storage 等依赖，在直连规则前强制走 `🧠 Claude Code` 组。两个代理组均只有 `PROXY` 和 `REJECT`，不含 `DIRECT`，也不自动轮换出口。完整官方网络要求：https://code.claude.com/docs/en/network-config 。
- DNS 与流量分流绑定：代理域名与备用 DNS 均使用经代理隧道的 Cloudflare/Google DoH（`#proxy`）；直连域名与节点启动解析使用阿里 IP 形式 DoH（`223.5.5.5`/`223.6.6.6`），且 `dns-direct-fallback-proxy = false` 禁止直连域名回退到代理 DNS。关闭系统 DNS 及其回退，劫持 IPv4 明文 DNS。
- HTTPDNS 分情况处理：放行美团、京东、微信、B站、微博、网易云音乐等国内直连服务的 HTTPDNS 以避免回退卡顿，其余未知 HTTPDNS 通过 `BlockHttpDNS.list` 拦截。
- IPv6 禁用并增加拒绝规则；拒绝应用 IPv4 UDP 任意端口，封锁 TCP 53/853、常见 STUN/TURN 端口（3478/5349/19302）及未知 `stun.`/`turn.` 域名（海外 AI 语音与主流通讯域名在前置走代理隧道以保证可用性）；私网 IP 段在本机直接 `REJECT`。
- 保留现有 YouTube 去广告重写和脚本；未将 Claude 域名加入 MITM。

## 导入后必须检查

1. 更新/重新导入上面的主配置，选中它；全局路由使用“配置”模式，首页选择实际代理节点。两个策略组都选 PROXY，不能使用全局直连。
2. 关闭会覆盖本配置 DNS/规则的其他模块，检查导入是否有不支持语法提示，并重新启动隧道。
3. 在设备可用时开启“包括所有网络 / Include All Networks”；它属于 App/系统 VPN 设置，不能只靠此配置文本代替。按需连接可帮助维持隧道，但不是系统防火墙。
4. 用请求日志检查 `api.anthropic.com`、`platform.claude.com`、`claude.ai` 等是否命中 Claude Code；测试一个 CN IP 和非 CN IP 的 TCP 连接。
5. **保持隧道运行，故意使所选节点不可达**，确认 Claude、国外网站及普通 DNS 均失败，日志无 DIRECT 回退；恢复节点后再测。另测隧道被关闭/崩溃的情况，后者不能由配置文件保证阻断。
6. 在 Wi-Fi、蜂窝及切换网络后分别做 DNS/WebRTC/IPv6 泄露测试，不应出现运营商 DNS 或 WebRTC 的真实公网地址。DNS 测试看到不同公共解析器出口 IP 不一定是泄露。

## 限制与代价

- 当前只能静态检查，尚未在用户设备上的 Shadowrocket 中验证解析、抓包或断线行为，不能据此宣称已经实现“零泄露”。
- 严格 UDP 拒绝可能影响微信/FaceTime 等音视频通话、WebRTC、游戏、局域网发现和 QUIC；有 TCP 替代的服务通常可回退。私网网段默认在本机直接拒绝（`REJECT`）。
- 直连域名使用国内阿里 DoH，代理域名使用经隧道的海外 DoH；直连 DNS 不设代理兜底，阿里 DoH 不可用时国内域名解析直接失败，不串到代理出口。
- WebRTC 的本地候选地址、任意端口的 TCP TURN 或指向 CN IP 的自定义服务无法仅靠这些网络规则彻底覆盖；浏览器支持时应禁用 WebRTC/非代理 ICE。若要保证任何应用绝不暴露真实公网 IP，就不能同时无条件允许其 CN 连接直连。
- 配置约束只对经过 Shadowrocket 的流量有效。手机 VPN 不自动代理电脑/WSL，也不保证热点客户端流量经过隧道。Claude Code 在电脑运行时，需要该电脑自己的 HTTP 代理设置或 TUN；官方支持 HTTP_PROXY/HTTPS_PROXY，不支持 SOCKS URL。自定义 ANTHROPIC_BASE_URL、第三方模型网关及 MCP 服务需另加其实际域名的强制代理规则。
- 隧道关闭、崩溃或系统绕过 VPN 时，“宁愿断网也不直连”需要 App/系统阻断或运行 Claude Code 的设备防火墙共同实现；配置文件无法单独提供系统级 kill switch。

## YouTube 去广告

适用于 iOS YouTube / YouTube Music App 的接口响应处理，不承诺浏览器网页端或所有 App 版本均有效。v2rayNG 的 JSON 文件仅提供 DNS/路由配置，不执行这些去广告脚本。

YouTube 去广告依赖 HTTPS 解密。导入配置后，需要在 Shadowrocket 中生成/安装 MITM CA 证书并在 iOS 中设为信任，否则脚本不会生效。

配置已覆盖 `youtubei.googleapis.com` 和 `youtubei-att.googleapis.com` 两个 API 主机。后者也可能被 YouTube App 使用。

排查顺序：

1. 从上面的 `shrc-revised.conf` 地址重新导入并选中配置；旧的 `shrc.conf` 与 `Shadowrocket.conf` 地址已停用。
2. 确认 HTTPS 解密已开启，CA 证书已安装，并在 iOS「设置 → 通用 → 关于本机 → 证书信任设置」中开启该证书的完全信任。
3. 更新 Shadowrocket 的外部脚本资源，彻底退出 YouTube 后重新打开。
4. 查看请求/脚本日志：`youtubei.googleapis.com` 或 `youtubei-att.googleapis.com` 的 `/youtubei/v1/player` 等响应应命中 `youtube.response`；若没有命中，先排查证书、QUIC、其他模块覆盖和实际请求主机。
5. 若脚本已命中但仍有广告，记录 Shadowrocket / YouTube 版本、广告类型和脚本错误日志。上游脚本基于 Surge 测试，不保证 Shadowrocket 或所有 YouTube 版本的兼容性。

不要为了去广告直接拒绝整个 `googlevideo.com`，否则正常视频也可能无法播放。

## 说明

DNS 泄露测试的结果还会受到节点、iOS 网络环境、应用自身 DoH/HTTPDNS、IPv6 等因素影响。配置只能尽量约束 DNS 路径，不能保证所有第三方测试页面都只显示同一个 DNS 出口。
