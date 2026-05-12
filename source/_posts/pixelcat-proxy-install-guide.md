---
title: PixelCat Proxy 安装指南：从连接服务器到客户端配置
date: 2026-05-12 10:00:00
description: PixelCat Proxy 一键部署教程，从 SSH 连接服务器、域名解析、防火墙放行，到安装 ForwardProxy 和 Hysteria2，并说明 HTTPS 伪装、端口跳跃、节点诊断工具与 GPLv3 开源协议。
tags:
  - PixelCat Proxy
  - ForwardProxy
  - Hysteria2
  - 服务器
  - 科学上网
---

PixelCat Proxy 是「像素猫 - 科学上网ICU」的一键部署脚本，适合在 Linux 服务器上部署和维护自建代理入口。它主要支持：

- **ForwardProxy**：NaiveProxy 兼容，走 `443/tcp`，由 Caddy 自动申请 HTTPS 证书。
- **Hysteria2**：走 QUIC/UDP，可开启端口跳跃，适合需要 UDP 代理和抗干扰能力的场景。
- **节点维护工具**：内置 BBR、IP 质量检测、流媒体解锁检测、网络质量 / 回程检测。

两种协议可以单独部署，也可以在同一台服务器、同一个域名上共存。

> 本文默认你已经有一台 VPS 和一个域名。请确保部署和使用符合所在地法律法规、服务商条款以及学校或公司的网络使用政策。

## 项目入口

- GitHub 项目：[PixelCatICU/pixelcat-proxy](https://github.com/PixelCatICU/pixelcat-proxy)
- 一键安装命令：

```bash
curl -fsSL https://raw.githubusercontent.com/PixelCatICU/pixelcat-proxy/main/install.sh | bash
```

- 支持协议：ForwardProxy 和 Hysteria2
- 节点维护：BBR、IP 质量检测、流媒体解锁检测、网络质量 / 回程检测
- 开源协议：GNU General Public License v3.0

## 最快安装路径

如果你已经准备好服务器、域名和防火墙端口，可以按这个最短流程先跑起来：

```bash
ssh root@你的服务器IP
curl -fsSL https://raw.githubusercontent.com/PixelCatICU/pixelcat-proxy/main/install.sh | bash
```

进入中文菜单后，先选 `1` 安装 ForwardProxy；如果还需要更强的 UDP 抗干扰能力，再选 `2` 安装 Hysteria2。

## 一、先看两个协议怎么选

如果你只想先跑起来，建议先装 **ForwardProxy**。它走标准 HTTPS 端口，浏览器直接访问代理域名时还能显示一个正常网站，整体更像普通 HTTPS 站点。

如果你的网络环境对 TCP 代理不友好，或者经常遇到限速、干扰、晚高峰抖动，可以再加装 **Hysteria2**。它走 QUIC/UDP，并且支持端口跳跃，抗 QoS 和抗封锁能力更强。

| 协议 | 主要优势 | 更适合 |
| --- | --- | --- |
| ForwardProxy | HTTPS 伪装自然，证书自动申请，和普通网站流量相似 | 日常网页、文档、视频、通用代理 |
| Hysteria2 | QUIC/UDP，支持端口跳跃，弱网和干扰环境下更稳 | 移动网络、跨境链路差、TCP 容易被限速的环境 |

PixelCat Proxy 也把常见节点维护命令放进同一个菜单里。部署完成后，可以直接用菜单检测服务器 IP 质量、流媒体解锁状态和网络回程，不需要再临时去找一堆脚本。

## 二、ForwardProxy 的优势：像一个正常 HTTPS 网站

ForwardProxy 使用 Caddy + `forwardproxy` 插件，客户端连接时走 `443/tcp`。这个端口本来就是 HTTPS 网站最常用的端口，所以从外观看，它不像一个裸露的奇怪服务。

它的重点优势有三点：

- **HTTPS 伪装自然**：Caddy 会自动申请 Let's Encrypt 证书，客户端通过 TLS 连接，外层表现和普通 HTTPS 连接一致。
- **浏览器访问有伪装站**：如果别人直接在浏览器里打开 `https://proxy.example.com`，看到的是你配置的伪装网站，而不是代理服务报错页。
- **运维简单**：证书自动续期，服务由 systemd 管理，配置集中保存在 `.env` 和 Caddyfile 里。

这里的伪装不是把代理变成完全不可识别，而是减少明显特征：端口是常见的 `443`，证书是真证书，域名能打开正常网页。对于日常使用，这比开一个陌生端口、返回异常页面的服务更自然。

## 三、Hysteria2 的优势：UDP、端口跳跃和抗封锁

Hysteria2 基于 QUIC/UDP。和传统 TCP 代理相比，它在丢包、抖动、弱网环境下通常更有韧性，尤其适合移动网络、跨境链路不稳定、晚高峰 QoS 明显的场景。

PixelCat Proxy 部署 Hysteria2 时支持 **UDP 端口跳跃**。默认可以让客户端在 `20000-50000/udp` 范围内发包，服务器再通过 nftables 或 iptables 把这些端口转发到实际监听端口。

端口跳跃的好处是：

- **降低单端口被持续干扰的影响**：不是所有流量都固定压在一个 UDP 端口上。
- **对抗简单端口限速**：如果网络只针对某个固定端口做 QoS，跳跃范围能减少影响。
- **更适合不稳定网络**：QUIC/UDP 本身对丢包和切换更友好，移动热点、跨运营商线路上通常更稳。

Hysteria2 也支持伪装 URL。普通探测访问时可以返回一个正常网站内容，不会直接暴露一个空服务。不过它的核心优势不是“像网页”，而是 UDP 传输、弱网性能和端口跳跃带来的抗干扰能力。

## 四、推荐部署组合

最稳妥的方式是两个都装：

1. 先安装 ForwardProxy，保证有一个标准 HTTPS 代理入口。
2. 再安装 Hysteria2，作为高干扰网络下的备用或主力线路。
3. 两个协议使用同一个域名：ForwardProxy 走 `443/tcp`，Hysteria2 走 `443/udp`，互不冲突。

这样你的客户端里可以同时保留两个出站：平时用 ForwardProxy，需要更强抗干扰时切到 Hysteria2。

## 五、准备服务器信息

开始之前，先准备好这几项：

| 项目 | 示例 | 说明 |
| --- | --- | --- |
| 服务器 IP | `1.2.3.4` | VPS 公网 IP |
| SSH 用户 | `root` | 也可以是有 sudo 权限的普通用户 |
| SSH 端口 | `22` | 如果改过端口，按实际填写 |
| 代理域名 | `proxy.example.com` | 后面用于申请证书和客户端连接 |
| 伪装域名 | `www.example.com` | 浏览器直接访问代理域名时反代显示的网站 |

服务器系统建议使用 Debian、Ubuntu、CentOS、Rocky Linux、AlmaLinux 或 Alpine，只要是 Linux + systemd 环境即可。

## 六、连接到服务器

在本地电脑打开终端，输入：

```bash
ssh root@1.2.3.4
```

如果你的 SSH 端口不是 `22`，例如是 `2222`，用：

```bash
ssh -p 2222 root@1.2.3.4
```

第一次连接时会看到指纹确认，输入：

```text
yes
```

然后输入服务器密码。登录成功后，终端一般会显示类似：

```text
root@server:~#
```

这就表示你已经进入服务器。

## 七、更新系统并安装基础工具

不同系统使用不同命令。Debian / Ubuntu 执行：

```bash
apt update
apt install -y curl tar
```

CentOS / Rocky Linux / AlmaLinux 执行：

```bash
yum install -y curl tar
```

如果是较新的系统，也可能使用：

```bash
dnf install -y curl tar
```

Alpine 执行：

```bash
apk add --no-cache curl tar
```

PixelCat Proxy 的安装入口会自动补齐常用依赖，但提前装好 `curl` 和 `tar` 可以减少第一次运行时的报错。

## 八、解析域名到服务器

进入你的域名 DNS 管理后台，添加一条记录：

```text
类型：A
主机记录：proxy
记录值：1.2.3.4
```

如果你用的是 IPv6，也可以添加 `AAAA` 记录。

解析完成后，在本地电脑或服务器上检查：

```bash
ping proxy.example.com
```

只要看到解析出的 IP 是你的服务器 IP，就可以继续。

如果你使用 Cloudflare 一类 CDN，首次部署建议先关闭代理云朵，保持 DNS only。证书申请和代理连接都需要真实连到你的服务器。

## 九、放行防火墙和安全组

ForwardProxy 至少需要：

```text
80/tcp
443/tcp
```

Hysteria2 至少需要：

```text
443/udp
20000-50000/udp
```

其中 `20000-50000/udp` 是默认端口跳跃范围，如果你部署时改了范围，就按实际范围放行。

注意要检查两个地方：

- 云厂商控制台的安全组 / 防火墙
- 服务器系统内部的防火墙，例如 `ufw`、`firewalld`、`iptables`

Ubuntu 如果启用了 `ufw`，可以执行：

```bash
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 443/udp
ufw allow 20000:50000/udp
ufw reload
```

如果没有启用系统防火墙，只需要在云厂商安全组里放行即可。

## 十、一键启动安装菜单

在服务器执行：

```bash
curl -fsSL https://raw.githubusercontent.com/PixelCatICU/pixelcat-proxy/main/install.sh | bash
```

这个命令会自动完成几件事：

1. 创建 `/opt/pixelcat` 目录。
2. 下载 `PixelCatICU/pixelcat-proxy` 项目代码。
3. 进入 `/opt/pixelcat/pixelcat-forwardproxy`。
4. 运行 `deploy.sh` 中文菜单。

如果之前已经安装过，再运行同一条命令会更新项目代码并重新打开菜单。

看到菜单后，大致是这样：

```text
1) 安装 / 更新 PixelCat ForwardProxy
2) 安装 / 更新 PixelCat Hysteria2
3) 卸载 PixelCat ForwardProxy
4) 卸载 PixelCat Hysteria2
5) 一键开启 BBR
6) IP 质量检测
7) 流媒体解锁检测
8) 网络质量 / 回程检测
0) 退出
```

## 十一、安装 ForwardProxy

如果你想先部署 NaiveProxy 兼容协议，输入：

```text
1
```

然后按提示填写：

| 提示 | 示例 | 说明 |
| --- | --- | --- |
| 代理域名 | `proxy.example.com` | 必须已经解析到服务器 |
| 代理用户名 | `pixelcat` | 客户端连接用 |
| 代理密码 | `change_this_password` | 建议使用强密码 |
| 伪装网站域名 | `www.example.com` | 只填域名，不要带 `https://` |
| 证书邮箱 | `admin@example.com` | 可留空 |
| HTTP 端口 | `80` | 默认即可 |
| HTTPS 端口 | `443` | 默认即可 |

脚本会自动：

- 下载或编译带 `forwardproxy` 插件的 Caddy。
- 创建 `pixelcat-proxy` 专用系统用户。
- 生成 Caddyfile。
- 自动申请 Let's Encrypt 证书。
- 创建并启动 `pixelcat-forwardproxy` systemd 服务。
- 输出 sing-box 客户端配置。

安装完成后，查看服务状态：

```bash
systemctl status pixelcat-forwardproxy --no-pager
```

查看实时日志：

```bash
journalctl -u pixelcat-forwardproxy -f
```

如果服务是 `active (running)`，说明服务已经启动。

## 十二、安装 Hysteria2

如果你想部署 Hysteria2，重新运行菜单：

```bash
cd /opt/pixelcat/pixelcat-forwardproxy
./deploy.sh
```

输入：

```text
2
```

然后按提示填写：

| 提示 | 示例 | 说明 |
| --- | --- | --- |
| Hysteria2 域名 | `proxy.example.com` | 可与 ForwardProxy 同域名 |
| Hysteria2 密码 | 留空 | 如果已有 ForwardProxy 配置，默认沿用 `.env` 里的 `PASSWORD`；否则自动生成强密码 |
| 监听 UDP 端口 | `443` | 默认即可 |
| 端口跳跃范围 | `20000-50000` | 输入 `off` 可关闭 |
| 上行限速 Mbps | `0` | `0` 表示不限速 |
| 下行限速 Mbps | `0` | `0` 表示不限速 |
| 伪装 URL | `https://www.bing.com` | 如果已有 ForwardProxy 配置，默认沿用 `https://DECOY_DOMAIN`；否则默认 `https://www.bing.com` |

如果你已经先安装了 ForwardProxy，并且只想让 Hysteria2 复用现有域名、密码和伪装站，可以直接执行：

```bash
./deploy.sh --install-hysteria2 -y
```

显式传入 `--hy2-domain`、`--hy2-password` 或 `--hy2-masquerade` 时，会覆盖从 ForwardProxy `.env` 读取到的默认值。

安装完成后检查：

```bash
systemctl status pixelcat-hysteria2 --no-pager
```

如果启用了端口跳跃，再检查：

```bash
systemctl status pixelcat-hysteria2-hop --no-pager
```

查看 Hysteria2 日志：

```bash
journalctl -u pixelcat-hysteria2 -f
```

## 十三、开启 BBR

BBR 可以改善部分线路的 TCP 传输表现。回到菜单：

```bash
cd /opt/pixelcat/pixelcat-forwardproxy
./deploy.sh
```

输入：

```text
5
```

或者直接执行：

```bash
./deploy.sh --bbr
```

检查是否开启成功：

```bash
sysctl net.ipv4.tcp_congestion_control
sysctl net.core.default_qdisc
```

正常会看到：

```text
net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq
```

## 十四、节点诊断工具

新版脚本把常用节点检测也放进了同一个菜单：

```text
6) IP 质量检测           (xykt/IPQuality)
7) 流媒体解锁检测         (lmc999/RegionRestrictionCheck)
8) 网络质量 / 回程检测     (xykt/NetQuality)
```

也可以直接用命令执行：

```bash
./deploy.sh --ip-quality
./deploy.sh --unlock-check
./deploy.sh --net-quality
```

这三个入口分别适合：

- **IP 质量检测**：看原生 IP、风险标签、黑名单和部分解锁状态。
- **流媒体解锁检测**：检查主流流媒体和区域解锁情况。
- **网络质量 / 回程检测**：检查延迟、测速和回程线路。

脚本会尽量自动安装 `jq`、`dig`、`mtr`、`iperf3`、`bc`、`imagemagick`、`nexttrace` 等检测依赖。网络质量和回程检测耗时会比较长，结果也会受测速点、BGP 数据源和路由探测可用性影响。

## 十五、配置客户端

部署结束后，脚本会在终端输出 sing-box 配置。请优先复制脚本实际输出的配置，因为里面包含你填写的域名、用户名、密码和端口。

ForwardProxy 的连接信息通常是：

```text
https://用户名:密码@代理域名
```

例如：

```text
https://pixelcat:change_this_password@proxy.example.com
```

sing-box 出站配置结构类似：

```json
{
  "type": "naive",
  "tag": "naive-out",
  "server": "proxy.example.com",
  "server_port": 443,
  "username": "pixelcat",
  "password": "change_this_password",
  "tls": {
    "enabled": true,
    "server_name": "proxy.example.com"
  }
}
```

Hysteria2 出站配置结构类似：

```json
{
  "type": "hysteria2",
  "tag": "hysteria-out",
  "server": "proxy.example.com",
  "server_port": 443,
  "server_ports": [
    "20000:50000"
  ],
  "password": "脚本生成或你自己填写的密码",
  "tls": {
    "enabled": true,
    "server_name": "proxy.example.com"
  },
  "up_mbps": 0,
  "down_mbps": 0
}
```

如果你关闭了端口跳跃，就不需要 `server_ports`。

## 十六、验证是否部署成功

先在浏览器打开：

```text
https://proxy.example.com
```

如果能看到伪装网站内容，说明 HTTPS 证书和 Caddy 基本正常。

再看服务状态：

```bash
systemctl status pixelcat-forwardproxy --no-pager
systemctl status pixelcat-hysteria2 --no-pager
```

最后用客户端连接代理，打开 IP 查询网站，确认出口 IP 已经变成服务器 IP。

如果连接失败，优先按这个顺序排查：

1. 域名是否解析到服务器 IP。
2. 云厂商安全组是否放行对应端口。
3. 服务器系统防火墙是否拦截。
4. `journalctl` 日志里是否有证书申请失败、端口占用或密码错误。
5. 客户端配置里的域名、端口、用户名、密码是否和脚本输出一致。

## 十七、更新、重启和卸载

重新打开菜单：

```bash
cd /opt/pixelcat/pixelcat-forwardproxy
./deploy.sh
```

更新 ForwardProxy 选 `1`，更新 Hysteria2 选 `2`。

重启 ForwardProxy：

```bash
systemctl restart pixelcat-forwardproxy
```

重启 Hysteria2：

```bash
systemctl restart pixelcat-hysteria2
```

卸载但保留配置：

```bash
./deploy.sh --uninstall
./deploy.sh --uninstall-hysteria2
```

彻底清理配置、证书和本地数据：

```bash
./deploy.sh --uninstall --purge
./deploy.sh --uninstall-hysteria2 --purge
```

彻底清理会删除配置和证书数据，执行前请确认自己不再需要这些信息。

## 常见问题

### 证书申请失败怎么办？

先确认 `proxy.example.com` 已经解析到服务器公网 IP，并且 `80/tcp`、`443/tcp` 已经放行。首次申请证书时，Let's Encrypt 必须能从公网访问到你的服务器。

### 伪装网站打不开怎么办？

`DECOY_DOMAIN` 只填域名，不要写 `https://`。另外，复杂网站可能因为 Cookie、CSP、WebSocket 或跨域限制导致反代显示不完整，建议使用简单官网、文档站或静态博客作为伪装站。

### Hysteria2 连不上怎么办？

重点检查 UDP。很多连接失败不是配置问题，而是云厂商安全组没有放行 `443/udp` 或端口跳跃范围。启用了端口跳跃时，`20000-50000/udp` 也要放行。

### 可以两个协议用同一个域名吗？

可以。ForwardProxy 使用 `443/tcp`，Hysteria2 使用 `443/udp`，同一个域名可以同时指向同一台服务器。

### 配置文件在哪里？

常用位置如下：

```text
/opt/pixelcat/pixelcat-forwardproxy/.env
/opt/pixelcat/pixelcat-forwardproxy/.env.hysteria2
/etc/pixelcat-forwardproxy/Caddyfile
/etc/pixelcat-hysteria2/config.yaml
/var/lib/pixelcat-forwardproxy
/var/lib/pixelcat-hysteria2
```

`.env` 和 `.env.hysteria2` 里有密码，不要公开截图或发给别人。

### Hysteria2 为什么有时会沿用 ForwardProxy 配置？

新版脚本会优先复用当前目录里 ForwardProxy `.env` 的配置：

- `HY2_DOMAIN` 默认沿用 `DOMAIN`
- `HY2_PASSWORD` 默认沿用 `PASSWORD`
- `HY2_MASQUERADE_URL` 默认沿用 `https://DECOY_DOMAIN`

这样两个协议可以更容易保持同域名、同密码、同伪装站。你也可以通过 `--hy2-domain`、`--hy2-password`、`--hy2-masquerade` 显式覆盖。

### 项目是什么开源协议？

PixelCat Proxy 当前使用 GNU General Public License v3.0。你可以自由使用、复制、修改和分发；如果分发修改版或衍生作品，需要继续按照 GPLv3 开源，并保留原版权声明和许可证文本。
