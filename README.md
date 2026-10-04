# shrc

个人 Shadowrocket 配置。

主配置：

```text
https://raw.githubusercontent.com/WANGXU-SU/shrc/refs/heads/main/shrc.conf
```

## 设计目标

- 基于 LingJingMaster/Shadowrocket-Rules 的 Shadowrocket.conf。
- 国内直连域名使用阿里 DNS / 腾讯 DNS 的 DoH。
- 代理域名使用 Cloudflare / Google DoH，并通过代理发送。
- 关闭系统 DNS fallback，配合 hijack-dns 和 BlockHttpDNS 规则减少 DNS 泄露。
- 默认关闭 IPv6。
- 集成 YouTube 去广告/增强脚本（Maasea/sgmodule）。

## YouTube 去广告

适用于 iOS YouTube / YouTube Music App 的接口响应处理，不承诺浏览器网页端或所有 App 版本均有效。v2rayNG 的 JSON 文件仅提供 DNS/路由配置，不执行这些去广告脚本。

YouTube 去广告依赖 HTTPS 解密。导入配置后，需要在 Shadowrocket 中生成/安装 MITM CA 证书并在 iOS 中设为信任，否则脚本不会生效。

配置已覆盖 `youtubei.googleapis.com` 和 `youtubei-att.googleapis.com` 两个 API 主机。后者也可能被 YouTube App 使用。

排查顺序：

1. 从上面的 `shrc.conf` 地址重新导入并选中配置；旧的 `Shadowrocket.conf` 地址已失效。
2. 确认 HTTPS 解密已开启，CA 证书已安装，并在 iOS「设置 → 通用 → 关于本机 → 证书信任设置」中开启该证书的完全信任。
3. 更新 Shadowrocket 的外部脚本资源，彻底退出 YouTube 后重新打开。
4. 查看请求/脚本日志：`youtubei.googleapis.com` 或 `youtubei-att.googleapis.com` 的 `/youtubei/v1/player` 等响应应命中 `youtube.response`；若没有命中，先排查证书、QUIC、其他模块覆盖和实际请求主机。
5. 若脚本已命中但仍有广告，记录 Shadowrocket / YouTube 版本、广告类型和脚本错误日志。上游脚本基于 Surge 测试，不保证 Shadowrocket 或所有 YouTube 版本的兼容性。

不要为了去广告直接拒绝整个 `googlevideo.com`，否则正常视频也可能无法播放。

## 说明

DNS 泄露测试的结果还会受到节点、iOS 网络环境、应用自身 DoH/HTTPDNS、IPv6 等因素影响。配置只能尽量约束 DNS 路径，不能保证所有第三方测试页面都只显示同一个 DNS 出口。
