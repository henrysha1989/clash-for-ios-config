# clash-for-ios-config

iOS 端 **Clash**（[ProjectClash/Clash-Client](https://github.com/ProjectClash/Clash-Client)，Hako-mihomo 内核）的分流配置。

> ⚠️ 本仓库**公开**，只放配置本身，**不含任何节点、订阅地址或凭据**。
> 节点请在客户端里用订阅导入，或在你本机的私有副本里填 —— **不要提交到本仓库**。

## 文件

| 文件 | 说明 |
|---|---|
| `config.yaml` | 主配置：TUN + fake-ip DNS + 8 个策略组 + 54 条分流规则 |

订阅地址（客户端里「配置 → 添加 → 从 URL 导入」）：

```
https://raw.githubusercontent.com/henrysha1989/clash-for-ios-config/main/config.yaml
```

国内直连超时可换自建加速前缀（`https://git.521989.xyz/` + 上面的完整地址）。
注意 raw 有 CDN 缓存，改完不一定立刻生效。

## 这份配置针对 iOS 内核做过什么

依据是内核仓库 `bind/clash/config_deviations.go`（对 `v1.19.32-clash.1` 核对，2026-10-11）。
iOS 端有一批键会被**平台强制或剥离**，写错值只会让 App 里多出「偏差提示」而没有实际效果。本配置的处理：

- `tun.auto-route` / `tun.auto-detect-interface` 在 iOS 上**强制 false**（路由由 App 用
  `NEPacketTunnelNetworkSettings` 安装），所以本文件写 `false`。
  **同一份文件拿到 PC / mihomo 上跑时，这两项要改回 `true`。**
- `keep-alive-idle` / `keep-alive-interval` **刻意不写**：内核 darwin 默认是
  5 分钟 / 75 秒（`component/keepalive/defaults_darwin.go`），显式写上游默认的 15 / 30
  会让蜂窝网络上的 TCP 保活探测过密、更费电。
- 以下键 iOS 端是强制值，本文件写的与之一致，因此**不会出现偏差提示**：
  `dns.enable: true`、`tun.enable: true`、`find-process-mode: off`、
  `profile.store-fake-ip: true`、`unified-delay: true`、`tun.stack: gvisor`
  （内核说明：不写 stack 也跑 gvisor，写 `gvisor`/`mips` 按写入值执行）。
- `geo-auto-update` / `geo-update-interval` / `geox-url` **由 App 读取并下载**，扩展自身不下载。
  App 已内置一份 geo 数据（`MetaCubeX/meta-rules-dat`），这里的 `geox-url` 指向自建加速前缀，
  供 App 侧更新使用。
- 规则集用 **MRS** 二进制格式，内核基于 mihomo 1.19.x，支持。
- iOS 上**不可用**、写了也会被忽略的：`easytier`、`faketcp`、`redir`、`tproxy`、
  `tun.include-*` / `tun.exclude-*`、`iptables`、`clash-for-android`。
- `allow-lan` 会被 App 的「局域网共享」开关覆盖（默认关）。本文件写 `false`，与开关一致。

## 要改的地方

配置里标了 **【个人定制区】** 的段落是环境相关的，换环境整段删掉或替换：

- 自建加速前缀（`geox-url`、规则集 URL）
- 公司域名直连（`megarobo.*`）
- 自建域名（`521989.xyz`）的直连与 fake-ip 过滤

`sniffer.skip-domain` 与 `rules` 里还有几处按需开关：

- WebRTC 截断（`DST-PORT,3478/5349`、`DOMAIN-KEYWORD,stun/turn`）会打断微信 / 腾讯会议 /
  Google Meet / Discord 的 P2P 音视频，要视频通话就把那 4 行注释掉。
- AI 分组 `AI 自动(美区)` 用节点名里的地区标记筛选美区节点；节点名不带地区标记时
  一个都匹配不到，组内会回退成 COMPATIBLE（≈直连）。

## 改动记录

- **2026-10-11**：首次入库。修掉两处失效引用 —— `dns.direct-nameserver` 与
  `dns.nameserver-policy.geosite:cn` 原来都指向自建 DoH `doh.521989.xyz`
  （该域名记录已于 2026-10-04 删除，实测 ENODATA），改成 `https://dns.alidns.com/dns-query`；
  同时删掉为它服务的 `nameserver-policy` 引导解析块。按 iOS 内核行为改
  `tun.auto-route` / `tun.auto-detect-interface` 为 `false`，去掉
  `keep-alive-idle` / `keep-alive-interval` 的显式值。
