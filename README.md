# shadowrocket-rules

## 1 生产级架构总图

```shell
        ┌────────────────────────┐
        │        iPhone         │
        │ Shadowrocket (L1)     │
        └─────────┬──────────────┘
                  ↓
        DNS (Cloudflare DoH)
                  ↓
        ┌────────────────────────┐
        │ AWS Tokyo EC2 (L2/L3) │
        │ BBR + MTU 1400        │
        │ KeepAlive tuned       │
        └─────────┬──────────────┘
                  ↓
        ┌────────────────────────┐
        │ Claude / OpenAI       │
        └────────────────────────┘
```

## 2 操作

* Step1: 将 [claude_production_stable.conf](./claude_production_stable.conf) 保存到手机 [文件]
* Step2: 将 claude_production_stable.conf 分享到 shadowrocket

## 3 订阅

shadowrocket -> 设置 -> 订阅

设置为：

```shell
打开时更新：关闭
自动后台更新：关闭
间隔：48小时
DNS：https://cloudflare-dns.com/dns-query, https://dns.google/dns-query
提醒：关闭
超时：10秒
根据Ping排序：关闭
```