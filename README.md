# clash-for-ios-config

iOS 端 **Clash**（[ProjectClash/Clash-Client](https://github.com/ProjectClash/Clash-Client)，Hako-mihomo 内核）的分流配置。

> ⚠️ 本仓库**公开**，只放配置本身，**不含任何节点、订阅地址或凭据**。
> 节点请在客户端里用订阅导入，或在你本机的私有副本里填 —— **不要提交到本仓库**。

## 文件

| 文件 | 说明 |
|---|---|
| `config.yaml` | 主配置：TUN + fake-ip DNS + 4 个策略组 + 42 条分流规则 + 7 个远程规则集 |
| `qr/accelerator.png` | 导入地址的二维码（扫码用） |

## 导入

客户端里「配置 → 添加 → 从 URL 导入」，填**加速地址**：

```
https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/clash-for-ios-config/main/config.yaml
```

![扫码导入](qr/accelerator.png)

扫码就是上面这条加速地址（600×600，反解验证过内容一致）。自建加速站走 GitHub Contents API，
响应头是 `cache-control: no-store` + `cf-cache-status: BYPASS` ⇒ **推完立刻可见**，
没有 raw 那种十几分钟 CDN 延迟。不想走加速站时，把开头的 `https://git.521989.xyz/` 去掉即可。

## 配置与节点分开管理

这个客户端把**配置**和**节点**分成两条独立管线：

1. 把本仓库的 `config.yaml` 作为**配置**添加（从 URL 导入）—— 它只提供分组与分流规则；
2. 节点在 App 的**「节点来源」**里单独添加（订阅 / 单节点 / 本地节点集），
   可以同时挂多个、逐个开关（至少保留一个）。

所以**本文件保持 `proxies: []`**，节点不写进来。分组能自动吃到 App 挂上来的节点，
是因为第 7 节的组用了 `include-all: true`。内核同时支持三种写法：

| 字段 | 收什么 |
|---|---|
| `include-all` | `proxies` + `proxy-providers`（本文件用这个） |
| `include-all-proxies` | 只收 `proxies` |
| `include-all-providers` | 只收 `proxy-providers` |

也可以用 `use: [名称]` 点名引用某个 provider。于是加节点、换订阅、开关某个节点源，
都不用动这个文件 —— 这也是它敢放在公开仓库的原因。

```yaml
    # 两个 include-all 组都带这条：滤掉机场塞在订阅里的"信息假节点"
    # （剩余流量 / 套餐到期 / 距离下次重置剩余 …）。它们测速还能"通过"，
    # 一旦被自动选择选中就是个废出口——只返回一个说明页。
    exclude-filter: '(?i)(剩余|到期|重置|官网|订阅|流量|距离|套餐|过期|expire|traffic|website|通知)'
```

> 桌面 mihomo（Clash Verge 等）没有这套「节点来源」，那时才需要在文件里自己配
> `proxies:` 或 `proxy-providers:`；**带订阅地址的副本不要推到本仓库**。

## 规则集

第 8 节有 7 个远程规则集：

| 类别 | 规则集 | 格式 | 来源 |
|---|---|---|---|
| 公开大表 | `BlockHttpDNS` / `Tesla` | `domain` + `mrs` | MetaCubeX/meta-rules-dat |
| **公开去广告表** | `AdBlockMihomo`（**233,137 条**） | `domain` + `mrs` | [217heidai/adblockfilters](https://github.com/217heidai/adblockfilters) 的 `adblockmihomolite.mrs` |
| **自建拦截表** | `ByteDanceAd` / `SoulAd` / `RejectCustom` | `classical` + `text` | [shadowrocket-adr-rules](https://github.com/henrysha1989/shadowrocket-adr-rules) 的 `clash-ios` 分支 |
| **自建直连表** | `DirectCustom` | `classical` + `text` | 同上（源表 `direct-custom.list`） |

> `AdBlockMihomo` 的换源史（名字一直跟源走）：
> 原来的 MetaCubeX geosite `category-ads-all.mrs` 实测只有 **902 条**
> （用 mihomo `GET /providers/rules` 读出来的 `ruleCount`），基本等于没拦。
> 2026-10-14 先换成 217heidai 的 **lite** 版（8,428 条 / 77 KB）。
> **2026-10-15 再换到完整版**（`adblockmihomo.mrs`，**233,137 条** / 2.2 MB）—— 理由是 owner 实测反馈
> **"Clash 的拦截效果比小火箭差很多"**：lite 只有 8,428 条，而小火箭那边挂着两张 4 万级表（合计 77,608），
> 完整版是 lite 的 **27.7 倍**、小火箭那两张合计的 **3 倍**。
> ✅ **内存实测（2026-10-15 01:25 真机，App「内核运行时」页）**：换成完整版后
> **内存占用 21.1 MB**（挂 lite 时是 20.8 MB ⇒ **只 +0.3 MB**），Go 堆 9.7（+1.8）、常驻内存 79.1（+3.4）。
> **50 MB 上限下毫无压力**，真机连续使用无异常。
> ⚠️ **估算方法的教训**：先前用本机 Linux mihomo `v1.19.32` 的 **RSS** 估过 +19.8 MB / 预计 ~40 MB，
> **高估约 6 倍** —— Linux RSS 含 Go 未归还的堆页；iOS 这边比的是 App 的「内存占用」，
> 加上 `with_low_memory` 内核的 trim/shed，真实增量只有 ~3 MB。
> **以后判断 iOS 内存一律真机读这一页，别拿 Linux RSS 外推。**
> **回滚**：`url` 换回 `adblockmihomolite.mrs`（一行）。

自建那四张表的**事实来源仍是该仓库 main 分支**的 Shadowrocket 格式清单
（`bytedance-ad.list` 263 条 / `soul-ad.list` 17 条 / `reject-custom.list` 20 条 /
`direct-custom.list` 121 条，共 421 条 —— **这几个数随上游同步浮动**，以 `clash-ios` 分支实际文件为准）。
`direct-custom.list` 是**与 Shadowrocket 共用同一张表**：手工区放公司域名（`megarobo.*`）等手改项，
自动区由 `sr_analyze.py` 从手机 DB 收集（抖音 / 腾讯等实测需直连的域）。
`clash-ios` 分支里是派生版本：**去掉第 3 列策略、丢掉注释行，类型与值逐字不动**
（即 `DOMAIN-SUFFIX,x,REJECT-DROP` → `DOMAIN-SUFFIX,x`）。动作由本配置决定：
拦截三行写 `RULE-SET,… ,REJECT`（想静默丢包就改 `REJECT-DROP`），直连那行写 `RULE-SET,… ,国内直连`。

main 一改，规则仓库的 Action（`.github/workflows/sync-clash-ios.yml` → `mihomo/sync.mjs`）
就会自动把四张表转好推到 `clash-ios` 分支，本地不用手动跑；想手动核对用 `node mihomo/sync.mjs --dry`。

**为什么自建表不用 MRS**：iOS 内核的 MRS 校验（`bind/clash/mrs_validation.go`）只支持
`domain` 与 `ipcidr` 两种 behavior，`classical` 会直接报
`MRS format is unsupported for provider behavior classical`；而这三张表是混合类型
（DOMAIN-SUFFIX / DOMAIN / DOMAIN-KEYWORD / IP-CIDR），装不进 `domain` 集。
288 条的规模也用不上 MRS 的性能收益 —— 那是给几万条的大表准备的。

规则顺序上：**直连优先于拦截**。公共去广告表在第 5 节、自建拦截表在第 6 节，
都排在**具体直连规则（第 3、4 节：内网 / 自家域 / 公司系统 / 钉钉 / Apple）之后**，
沿用规则仓库的既有约定 —— 自家域名与直连优先，不让"拦广告"误伤正常访问；
同时两节都在最后的 `GEOSITE,CN` / `GEOIP,CN` 国内兜底**之前**，否则 CN 广告域会被兜底放走。

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
- **IPv6 必须开着**（`ipv6: true` + `dns.ipv6: true`）。内核 `parseIPV6()` 在 `ipv6: false` 时会把
  `tun.inet6-address` 清空，App 拿不到 v6 地址就声明 `tun: IPv6 declaration applied=false`
  ——**隧道里没有 IPv6**。而手机的网络（Wi-Fi / 蜂窝）是有 v6 的，于是 **IPv6 流量与 v6 DNS
  全部走物理链路**，被运营商解析；browserleaks 能同时看到本配置的 DoH（Cloudflare）和运营商解析器。
  本文件因此写：`ipv6: true`、`dns.ipv6: true`、`dns.fake-ip-range6: fdfe:dcba:9876::1/64`、
  `tun.inet6-address: [fdfe:dcba:9877::1/126]`（v6 接口段与假 IP 池分开，取内核官方示例的值）。
  改完自检：App 日志里应出现 `tun: IPv6 declaration applied=true`。

## 要改的地方

配置里标了 **【个人定制区】** 的段落是环境相关的，换环境整段删掉或替换：

- 自建加速前缀（`geox-url`、规则集 URL）
- 公司域名直连（`megarobo.*`）
- 自建域名（`521989.xyz`）的直连与 fake-ip 过滤

`sniffer.skip-domain` 与 `rules` 里还有几处按需开关：

- WebRTC 截断（`DST-PORT,3478/5349`、`DOMAIN-KEYWORD,stun/turn`）会打断微信 / 腾讯会议 /
  Google Meet / Discord 的 P2P 音视频，要视频通话就把那 4 行注释掉。

> 原来的 `AI 美国` / `AI 自动(美区)` 两个分组已删除（2026-10-11）。原因：`AI 自动(美区)` 靠
> **节点名里的地区标记**筛选美区节点，而订阅节点名不带标记时一个都匹配不到，App 里会显示成
> 「URLTEST · 无节点」，组内只剩 `COMPATIBLE`（≈直连）——**等于把 Gemini 送去直连**，比没有这个组更糟。
> Gemini 随后改指 `节点选择`；**2026-10-14 连 Gemini 规则集一起删了**（见改动记录），
> AI 段目前是空的 —— 要接 OpenAI / Claude / Perplexity 时在那里加规则。

## 改动记录

- **2026-10-14**：**删掉 `Gemini` 规则集**（owner 指令）。它是一张 blackmatrix7 的 classical 文本表
  （13 条），但**内容已被 `GEOSITE,google` 完全覆盖**：13 条里 10 条域名/后缀规则本就落在
  `+.google.com` / `+.googleapis.com` / `+.google.dev` / `+.google` / `+.deepmind.com` 里；
  剩下 3 条 `DOMAIN-KEYWORD`（`colab` / `developerprofiles` / `generativelanguage`）的真实主机
  （`generativelanguage.googleapis.com`、`colab.*.googleusercontent.com`、
  `developerprofiles-pa.clients6.google.com`）也全在 geosite google 覆盖内。
  两条兜底的 `DOMAIN-SUFFIX`（`gemini.google.com` / `aistudio.google.com`）同样由 `GEOSITE,google` 接住。
  ⇒ **零覆盖损失**，少一个远程依赖（那张表正是 2026-10-11「Gemini: TLS 错误导致安全连接失败」的来源）。
  规则 45 → **42** 条、规则集 8 → **7** 个，后续章节顺延重编号。
- **2026-10-14**：**换掉公共去广告表 + 挪顺序**。`Advertising_Domain`（MetaCubeX geosite
  `category-ads-all.mrs`，实测只有 **902 条**）换成 **`AdBlockMihomoLite`**
  （217heidai `adblockmihomolite.mrs`，**8,428 条**，每 8h 更新）—— **规则集名字跟源走**，
  不再沿用 geosite 时代的 `Advertising_Domain`。**不动**自建三表。同时把这条 `RULE-SET` 从第 1 节
  挪到第 5 节（具体直连之后、`GEOSITE,CN` 之前），给自家域/公司系统留逃生路径 ——
  新表是合并表，比 geosite 激进。规则总数 45 条、规则集 8 个都不变，只是顺序变了。
  为什么不用完整版（233,137 条）：mihomo `v1.19.32` 实测加载后 RSS +19.8 MB，iOS 扩展 50 MB 上限扛不住；
  lite 版只 +2.3 MB。选型与实测见 `research/去广告规则集替换-评估-20261014.md`。
- **2026-10-15**：**去广告表 lite → 完整版**。`AdBlockMihomoLite`(8,428) → **`AdBlockMihomo`(233,137)**，
  名字跟着源走（`adblockmihomolite.mrs` → `adblockmihomo.mrs`），缓存路径同步改
  `./rules/adblockmihomo.mrs`。起因：owner 实测反馈 Clash 的拦截明显弱于小火箭。
  ⚠️ 内存 +19.8 MB（实测），iOS 50 MB 上限下需真机验收；回滚 = 一行 url。
  规则总数 42 条、规则集 8 个都不变（只换源与名字）。
  **真机内存实测：20.8 → 21.1 MB（+0.3 MB，App「内核运行时」页 01:25）**、常驻 +3.4 MB ⇒ 50 MB 上限下无压力；
  先前用 Linux RSS 估的「~40 MB」高估了约 6 倍（教训已写进 README 正文与配置注释）。
- **2026-10-15**：**自建拦截表 +7 条**（owner 抓包驱动，全部进 `reject-custom.list` 的**手工区** ——
  该区人工维护，`sr_analyze.py` 只收敛 AUTO 标记以下、`clash-ios` 分支自动同步 ⇒ 小火箭与 Clash 两端同时生效）。
  ① Soul 抓包 ⇒ 腾讯优量汇（GDT）/ 腾讯广告 SDK 共 2 条；
  ② 探探 + 高德抓包 ⇒ 阿里妈妈 Tanx（曝光/点击信标）、秒针监测、优酷广告 ATM、AppsFlyer 归因 4 条后缀，
  外加高德广告素材域 1 条。**探探开屏本身无需新规** —— 它的 `/v2/ad/configuration` 明确是 GDT 竞价（①已覆盖）。
  口径与取舍（含「为什么不拦 alicdn / 探探主 API / 路径级为何不做」）见工作区
  `ops/clash/tantan-amap-HAR-分析-20261015.md`；分流侧无需改规则，`RULE-SET,RejectCustom` 已在。
  `RejectCustom` 13 → **20** 条，自建三表合计 293 → **300** 条（`ByteDanceAd` 263 / `SoulAd` 17 / `RejectCustom` 20）。
- **2026-10-12**：**自定义直连规则改为引用自建直连表**。原来散在规则里的
  `DOMAIN-SUFFIX,megarobo.com/.tech/.info,国内直连` 换成一条 `RULE-SET,DirectCustom,国内直连`，
  接 `shadowrocket-adr-rules` 的 `clash-ios` 分支新增的 `mihomo/direct-custom.list`（122 条）——
  与 Shadowrocket 那份 `direct-custom.list` **同一张表**，改表两端同时生效。
  `DOMAIN-SUFFIX,521989.xyz`（自建加速站域名）**仍留在配置里内联**，没进共享表。
  为此把 `mihomo/sync.mjs` 的 `FILES` 与同步 workflow 的 `paths` 都补上了 `direct-custom.list`。
- **2026-10-11**：精简分组 + 过滤无效节点。删掉两个**默认朝向与 `节点选择` 相同**的冗余间接层：
  `国外代理`（16 条规则改指 `节点选择`）和 `漏网之鱼`（`MATCH` 改指 `节点选择`），默认行为不变，
  **策略组 8 → 4**：节点选择 / 手动选择 / 自动选择 / 国内直连。
  保留 `国内直连`：它的默认朝向是 `DIRECT`（与 `节点选择` 相反），被 21 条规则引用，是"一键把国内流量
  切去代理"的开关，不是重复。
  给 `手动选择`、`自动选择` 加 `exclude-filter`，按名字滤掉机场的信息假节点
  （拿 50 个真实节点名验证：精确排除 4 个假节点、46 个真节点零误伤）。
- **2026-10-11**：删除 `AI 美国` / `AI 自动(美区)` 两个分组（App 里显示「URLTEST · 无节点」，
  地区 filter 匹配不到任何节点 ⇒ 组内只剩 COMPATIBLE≈直连，反而把 Gemini 送去直连）。
  Gemini 的三条规则改指 `国外代理`。分组 8 → 6 个。
- **2026-10-11**：Gemini 规则集从 `cdn.jsdelivr.net` 直连改走自建加速前缀（jsdelivr 在国内
  TLS 握手失败，App 里报「Gemini: TLS 错误导致安全连接失败」）。**内容不变**（blackmatrix7 同仓库同文件，
  只换投递路径）。至此规则集全部走加速站 —— 第三方 CDN 在国内不可靠，新加规则集时别用。
- **2026-10-11**：**修 IPv6 泄漏**。原文件 `ipv6: false` + `dns.ipv6: false`（继承自模板），
  导致内核清空 `tun.inet6-address`、App 声明 `applied=false`，隧道不含 v6 ⇒ 手机的 IPv6 流量与
  v6 DNS 全走物理链路（browserleaks 实测混入运营商解析器）。改为 `ipv6: true` + `dns.ipv6: true`，
  并显式给出 `dns.fake-ip-range6: fdfe:dcba:9876::1/64` 与 `tun.inet6-address: [fdfe:dcba:9877::1/126]`。
- **2026-10-11**：AI 段只保留 Gemini（删掉 OpenAI / ChatGPT / Claude / Perplexity / x.ai /
  Midjourney / Poe 等内联域名）；新增第 5 节「自建拦截表」，接 `shadowrocket-adr-rules` 的
  `clash-ios` 分支（`ByteDanceAd` 263 条 / `SoulAd` 17 条 / `RejectCustom` 13 条，classical 文本，条数随同步浮动）。
- **2026-10-11**：按客户端「配置与节点分开管理」的机制整理第 6 节与本文档 —— 本文件只当配置用，
  节点交给 App 的「节点来源」，`proxies: []` 保持空，分组靠 `include-all` 自动接收。
- **2026-10-11**：首次入库。修掉两处失效引用 —— `dns.direct-nameserver` 与
  `dns.nameserver-policy.geosite:cn` 原来都指向自建 DoH `doh.521989.xyz`
  （该域名记录已于 2026-10-04 删除，实测 ENODATA），改成 `https://dns.alidns.com/dns-query`；
  同时删掉为它服务的 `nameserver-policy` 引导解析块。按 iOS 内核行为改
  `tun.auto-route` / `tun.auto-detect-interface` 为 `false`，去掉
  `keep-alive-idle` / `keep-alive-interval` 的显式值。
