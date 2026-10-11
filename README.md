# clash-for-ios-config

iOS 端 Clash / mihomo 系客户端的配置文件存放处。

> ⚠️ 本仓库**只有配置骨架与说明**，不含任何节点、订阅地址或凭据。
> 节点请用客户端自带的订阅导入，或在本机私有副本里填，**不要提交到本仓库**。

## 文件

| 文件 | 说明 |
|---|---|
| `config.yaml` | Clash / mihomo 配置骨架（占位，规则与节点待填） |

## 用法

1. 客户端里用「从 URL 导入」拉 `config.yaml` 的 raw 地址；
2. 或在客户端内置的配置编辑器里粘贴内容后再填自己的节点。

raw 地址：

```
https://raw.githubusercontent.com/henrysha1989/clash-for-ios-config/main/config.yaml
```

（如直连超时，可用自建加速前缀 `https://git.521989.xyz/`。注意 raw 有 CDN 缓存，改完客户端不一定立刻生效。）

## 注意

- 本仓库**公开**，任何写进 `config.yaml` 的内容都是公开的 —— 节点、UUID、密码一律不要进门。
- 规则集请引用上游公开列表，不要在这里放自建清单的私密 URL。
