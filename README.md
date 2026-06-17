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

## 4 AWS EC2 设置

### 4.1 TCP keepalive

修改 /etc/sysctl.conf

```shell
vim  /etc/sysctl.conf
```

新增：

```shell
net.ipv4.tcp_keepalive_time = 120
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 5
```

使生效：

```shell
sudo sysctl -p
```

### 4.2 MTU 1400 

修改 /etc/netplan

```shell
ls /etc/netplan
```

比如显示：50-cloud-init.yaml，修改

```shell
vim /etc/netplan/50-cloud-init.yaml 
```

写入：

```shell
network:
  version: 2
  ethernets:
    <your env>:
      match:
        macaddress: "<your-macaddress>"
      set-name: "ens5"
      dhcp4: true
      dhcp6: false
      mtu: 1400
```

使得生效：

```shell
sudo netplan apply
```

查询：

```shell
ip link show <your env>
```

### 4.3 BBR

#### 查看系统版本

```shell
cat /etc/os-release
```

PRETTY_NAME="Ubuntu 24.04.4 LTS"

#### 修改 `/etc/modules-load.d/bbr.conf`

```shell
sudo vim /etc/modules-load.d/bbr.conf
```

写入：

```shell
tcp_bbr
```

#### 修改 `/etc/sysctl.conf`，新增

```shell
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

使生效：

```shell
sudo sysctl -p
```

#### 启动 tcp_bbr

```
sudo modprobe tcp_bbr
```

#### 查询 BBR

```shell
sysctl net.ipv4.tcp_congestion_control
```

net.ipv4.tcp_congestion_control = bbr

```shell
lsmod | grep bbr
```

tcp_bbr                20480  15

看到这两条输出，说明：

- BBR 内核模块已加载
- fq 调度器已启用
- congestion control 已切换为 BBR
