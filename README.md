# shrc

个人 Shadowrocket 配置。

主配置：

```text
https://raw.githubusercontent.com/WANGXU-SU/shrc/refs/heads/main/Shadowrocket.conf
```

## 设计目标

- 基于 LingJingMaster/Shadowrocket-Rules 的 Shadowrocket.conf。
- 国内直连域名使用阿里 DNS / 腾讯 DNS 的 DoH。
- 代理域名使用 Cloudflare / Google DoH，并通过代理发送。
- 关闭系统 DNS fallback，配合 hijack-dns 和 BlockHttpDNS 规则减少 DNS 泄露。
- 默认关闭 IPv6。
- 集成 YouTube 去广告/增强脚本（Maasea/sgmodule）。

## YouTube 去广告

YouTube 去广告依赖 HTTPS 解密。导入配置后，需要在 Shadowrocket 中生成/安装 MITM CA 证书并在 iOS 中设为信任，否则脚本不会生效。

MITM 仅配置 YouTube/Google Video 相关域名，尽量缩小解密范围。

## 说明

DNS 泄露测试的结果还会受到节点、iOS 网络环境、应用自身 DoH/HTTPDNS、IPv6 等因素影响。配置只能尽量约束 DNS 路径，不能保证所有第三方测试页面都只显示同一个 DNS 出口。
