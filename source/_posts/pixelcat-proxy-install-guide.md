---
title: PixelCat Proxy 安装指南：NaiveProxy 与 Hysteria2 一键部署
date: 2026-05-12 10:00:00
updated: 2026-05-14 16:30:00
lang: zh-CN
translation_key: pixelcat-proxy-install-guide
translations:
  zh-CN: /posts/pixelcat-proxy-install-guide/
  ru: /ru/posts/pixelcat-proxy-install-guide/
  fa: /fa/posts/pixelcat-proxy-install-guide/
description: PixelCat Proxy 一键部署教程，从 SSH 连接服务器、域名解析、防火墙放行，到安装 NaiveProxy 和 Hysteria2，并说明 HTTPS 伪装、端口跳跃、预编译 Caddy、证书复用、节点诊断工具与 GPLv3 开源协议。
tags:
  - PixelCat Proxy
  - NaiveProxy
  - Hysteria2
  - 服务器
  - 科学上网
---

PixelCat Proxy 是「像素猫 - 科学上网ICU」面向 Linux 服务器的一键部署脚本，适合部署和维护自建代理入口。它主要支持：

- **NaiveProxy**：使用 NaiveProxy 作者维护的 `github.com/klzgrad/forwardproxy` Caddy 插件，由 Caddy 提供 TLS、HTTP/2 CONNECT、Basic Auth、probe resistance 和伪装站点。
- **Hysteria2**：使用官方 Hysteria2 二进制，走 QUIC/UDP，可开启端口跳跃。
- **节点维护工具**：内置 BBR、IP 质量检测、流媒体解锁检测、网络质量 / 回程检测。

两种协议可以单独部署，也可以在同一台服务器、同一个域名上共存：NaiveProxy 走 `443/tcp`，Hysteria2 走 `443/udp`。

> 本文默认你已经有一台 VPS 和一个域名。请确保部署和使用符合所在地法律法规、服务商条款以及学校或公司的网络使用政策。

## 项目入口

- GitHub 项目：[PixelCatICU/pixelcat-proxy](https://github.com/PixelCatICU/pixelcat-proxy)
- 官网：[pixelcat.icu](https://pixelcat.icu)
- YouTube：[@PixelCatICU](https://www.youtube.com/@PixelCatICU)
- 一键安装命令：

```bash
curl -fsSL https://raw.githubusercontent.com/PixelCatICU/pixelcat-proxy/main/install.sh | bash
```

- 支持协议：NaiveProxy 和 Hysteria2
- 节点维护：BBR、IP 质量检测、流媒体解锁检测、网络质量 / 回程检测
- 开源协议：GNU General Public License v3.0

## 最快安装路径

如果你已经准备好服务器、域名和防火墙端口，可以按这个最短流程先跑起来：

```bash
ssh root@你的服务器IP
curl -fsSL https://raw.githubusercontent.com/PixelCatICU/pixelcat-proxy/main/install.sh | bash
```

进入菜单后先选择中文，再选 `1` 安装 NaiveProxy；如果还需要更强的 UDP 抗干扰能力，再选 `2` 安装 Hysteria2。脚本也支持俄语和波斯语，可用 `--lang ru` 或 `--lang fa` 直接指定。

## 一、先看两个协议怎么选

如果你只想先跑起来，建议先装 **NaiveProxy**。它走标准 HTTPS 端口，浏览器直接访问代理域名时还能显示一个正常网站，整体更像普通 HTTPS 站点。

如果你的网络环境对 TCP 代理不友好，或者经常遇到限速、干扰、晚高峰抖动，可以再加装 **Hysteria2**。它走 QUIC/UDP，并且支持端口跳跃，抗 QoS 和抗封锁能力更强。

| 协议 | 主要优势 | 更适合 |
| --- | --- | --- |
| NaiveProxy | HTTPS 伪装自然，证书自动申请，HTTP/2 CONNECT，probe resistance，和普通网站流量相似 | 日常网页、文档、视频、通用代理 |
| Hysteria2 | QUIC/UDP，支持端口跳跃，弱网和干扰环境下更稳 | 移动网络、跨境链路差、TCP 容易被限速的环境 |

PixelCat Proxy 也把常见节点维护命令放进同一个菜单里。部署完成后，可以直接用菜单检测服务器 IP 质量、流媒体解锁状态和网络回程，不需要再临时去找一堆脚本。

## 二、NaiveProxy 的优势：像一个正常 HTTPS 网站

PixelCat Proxy 当前部署的是 NaiveProxy 兼容服务端：带 `klzgrad/forwardproxy@naive` 插件的 Caddy。它不是独立的 `naive` 二进制，而是把 NaiveProxy padding layer 放进 Caddy 的 forward_proxy 处理器里。

上游构建方式与 NaiveProxy 官方 README 的 Caddy 示例一致：

```bash
xcaddy build \
  --with github.com/caddyserver/forwardproxy=github.com/klzgrad/forwardproxy@naive
```

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

1. 先安装 NaiveProxy，保证有一个标准 HTTPS 代理入口。
2. 再安装 Hysteria2，作为高干扰网络下的备用或主力线路。
3. 两个协议使用同一个域名：NaiveProxy 走 `443/tcp`，Hysteria2 走 `443/udp`，互不冲突。

这样你的客户端里可以同时保留两个出站：平时用 NaiveProxy，需要更强抗干扰时切到 Hysteria2。

## 五、准备服务器信息

开始之前，先准备好这几项：

| 项目 | 示例 | 说明 |
| --- | --- | --- |
| 服务器 IP | `1.2.3.4` | VPS 公网 IP |
| SSH 用户 | `root` | 也可以是有 sudo 权限的普通用户 |
| SSH 端口 | `22` | 如果改过端口，按实际填写 |
| 代理域名 | `proxy.example.com` | 后面用于申请证书和客户端连接 |
| 伪装域名 | `www.example.com` | 浏览器直接访问代理域名时反代显示的网站 |

服务器系统建议使用 Debian、Ubuntu、RHEL 系、Fedora、Alpine 等常见 Linux 发行版，并且需要 systemd。脚本支持 `amd64` 和 `arm64`。

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

NaiveProxy 至少需要：

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
3. 进入 `/opt/pixelcat/pixelcat-naiveproxy`。
4. 运行 `deploy.sh` 三语菜单。

如果之前已经安装过，再运行同一条命令会更新项目代码并重新打开菜单。

指定语言：

```bash
./deploy.sh --lang zh   # 中文
./deploy.sh --lang ru   # Русский
./deploy.sh --lang fa   # فارسی
```

看到菜单后，大致是这样：

```text
1) 安装 / 更新 PixelCat NaiveProxy
2) 安装 / 更新 PixelCat Hysteria2
3) 卸载 PixelCat NaiveProxy
4) 卸载 PixelCat Hysteria2
5) 一键开启 BBR
6) IP 质量检测
7) 流媒体解锁检测
8) 网络质量 / 回程检测
0) 退出
```

查看全部参数：

```bash
cd /opt/pixelcat/pixelcat-naiveproxy
./deploy.sh --help
```

## 十一、安装 NaiveProxy

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

- 优先下载预编译 `pixelcat-naiveproxy-caddy`，并校验 `.sha256`。
- 如果预编译包不可用、校验失败或缺少 `forward_proxy` 模块，则回退到本机安装 Go 和 xcaddy 编译。
- 创建 `pixelcat-naiveproxy` 专用 Caddy 服务。
- 生成 `/etc/pixelcat-naiveproxy/Caddyfile`。
- 自动申请 Let's Encrypt 证书。
- 创建并启动 `pixelcat-naiveproxy.service`。
- 输出 sing-box 客户端配置。

非交互安装示例：

```bash
./deploy.sh --install -y \
  --domain proxy.example.com \
  --username pixelcat \
  --password change_this_password \
  --decoy-domain www.example.com \
  --email admin@example.com
```

生产环境更推荐交互式输入密码，因为 `--password` 可能被 shell 历史或进程列表记录。

如果你想强制本地编译 Caddy：

```bash
./deploy.sh --install --build-from-source
```

如果你只想生成配置、暂时不启动服务：

```bash
./deploy.sh --install --skip-start
```

安装完成后，查看服务状态：

```bash
systemctl status pixelcat-naiveproxy --no-pager
```

查看实时日志：

```bash
journalctl -u pixelcat-naiveproxy -f
```

确认 Caddy 是否带有 forward_proxy 模块：

```bash
/usr/local/bin/pixelcat-naiveproxy-caddy list-modules | grep forward_proxy
```

如果服务是 `active (running)`，说明服务已经启动。

## 十二、安装 Hysteria2

如果你想部署 Hysteria2，重新运行菜单：

```bash
cd /opt/pixelcat/pixelcat-naiveproxy
./deploy.sh
```

输入：

```text
2
```

然后按提示填写：

| 提示 | 示例 | 说明 |
| --- | --- | --- |
| Hysteria2 域名 | `proxy.example.com` | 如果已有 NaiveProxy 配置，默认沿用 `.env` 里的 `DOMAIN` |
| Hysteria2 密码 | 留空 | 如果已有 NaiveProxy 配置，默认沿用 `.env` 里的 `PASSWORD`；否则自动生成强密码 |
| 监听 UDP 端口 | `443` | 默认即可 |
| 端口跳跃范围 | `20000-50000` | 输入 `off` 可关闭 |
| 端口跳跃网卡 | 留空 | 默认自动检测默认路由网卡，也可用 `--hy2-hop-iface` 指定 |
| 上行限速 Mbps | `0` | `0` 表示不限速 |
| 下行限速 Mbps | `0` | `0` 表示不限速 |
| 伪装 URL | `https://www.bing.com` | 如果已有 NaiveProxy 配置，默认沿用 `https://DECOY_DOMAIN`；否则默认 `https://www.bing.com` |

如果你已经先安装了 NaiveProxy，并且只想让 Hysteria2 复用现有域名、密码和伪装站，可以直接执行：

```bash
./deploy.sh --install-hysteria2 -y
```

显式传入 `--hy2-domain`、`--hy2-password`、`--hy2-hop-iface` 或 `--hy2-masquerade` 时，会覆盖默认值。

关闭端口跳跃：

```bash
./deploy.sh --install-hysteria2 -y --hy2-hop-range off
```

只写入配置、不启动服务：

```bash
./deploy.sh --install-hysteria2 --skip-start
```

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

Hysteria2 的证书逻辑也做了兼容：

- 如果 `/var/lib/pixelcat-naiveproxy` 下已经有同域名证书，Hysteria2 会直接复用。
- 如果没有可复用证书，Hysteria2 会使用自己的 ACME 配置申请证书。
- Hysteria2 自申请 ACME 时需要 `443/tcp` 可用于 TLS-ALPN-01 校验。
- 先安装 Hysteria2、后安装 NaiveProxy 也可以；同域名场景下后续会优先复用 Caddy 证书。

## 十三、开启 BBR

BBR 可以改善部分线路的 TCP 传输表现。回到菜单：

```bash
cd /opt/pixelcat/pixelcat-naiveproxy
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

脚本会写入 `/etc/sysctl.d/99-pixelcat-bbr.conf`。检查是否开启成功：

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

脚本把常用节点检测也放进了同一个菜单：

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

NaiveProxy 的连接信息通常是：

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
systemctl status pixelcat-naiveproxy --no-pager
systemctl status pixelcat-hysteria2 --no-pager
```

如果启用了端口跳跃，可以看 NAT 规则：

```bash
nft list table inet pixelcat-hy 2>/dev/null || iptables -t nat -L PREROUTING -n
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
cd /opt/pixelcat/pixelcat-naiveproxy
./deploy.sh
```

更新 NaiveProxy 选 `1`，更新 Hysteria2 选 `2`。

重启 NaiveProxy：

```bash
systemctl restart pixelcat-naiveproxy
```

重启 Hysteria2：

```bash
systemctl restart pixelcat-hysteria2
```

卸载 NaiveProxy 转发配置，但保留 Caddy 服务、证书目录和 Hysteria2 数据：

```bash
./deploy.sh --uninstall
```

同时删除当前项目目录下的 NaiveProxy `.env`：

```bash
./deploy.sh --uninstall --purge
```

卸载 Hysteria2，但保留配置和证书数据：

```bash
./deploy.sh --uninstall-hysteria2
```

彻底清理 Hysteria2 配置、证书数据和 `.env.hysteria2`：

```bash
./deploy.sh --uninstall-hysteria2 --purge
```

彻底清理会删除配置、证书或本地环境文件，执行前请确认自己不再需要这些信息。

## 十八、文件位置

NaiveProxy 常用位置：

```text
/usr/local/bin/pixelcat-naiveproxy-caddy
/etc/pixelcat-naiveproxy/Caddyfile
/var/lib/pixelcat-naiveproxy
/etc/systemd/system/pixelcat-naiveproxy.service
/opt/pixelcat/pixelcat-naiveproxy/.env
```

Hysteria2 常用位置：

```text
/usr/local/bin/pixelcat-hysteria2
/etc/pixelcat-hysteria2/config.yaml
/etc/pixelcat-hysteria2/hop-up.sh
/etc/pixelcat-hysteria2/hop-down.sh
/var/lib/pixelcat-hysteria2
/etc/systemd/system/pixelcat-hysteria2.service
/etc/systemd/system/pixelcat-hysteria2-hop.service
/etc/sysctl.d/99-pixelcat-hysteria2.conf
/opt/pixelcat/pixelcat-naiveproxy/.env.hysteria2
```

`.env` 和 `.env.hysteria2` 里有密码，不要公开截图或发给别人。

## 常见问题

### 证书申请失败怎么办？

先确认 `proxy.example.com` 已经解析到服务器公网 IP，并且 `80/tcp`、`443/tcp` 已经放行。首次申请证书时，Let's Encrypt 必须能从公网访问到你的服务器。

如果只装 Hysteria2 且没有可复用的 Caddy 证书，Hysteria2 自申请 ACME 时也需要 `443/tcp` 可用于 TLS-ALPN-01 校验。

### 伪装网站打不开怎么办？

`DECOY_DOMAIN` 只填域名，不要写 `https://`。另外，复杂网站可能因为 Cookie、CSP、WebSocket 或跨域限制导致反代显示不完整，建议使用简单官网、文档站或静态博客作为伪装站。

### Hysteria2 连不上怎么办？

重点检查 UDP。很多连接失败不是配置问题，而是云厂商安全组没有放行 `443/udp` 或端口跳跃范围。启用了端口跳跃时，`20000-50000/udp` 也要放行。

如果脚本提示无法自动检测默认网卡，可以用 `--hy2-hop-iface` 指定，或者把端口跳跃范围输入为 `off` 先关闭。

### 可以两个协议用同一个域名吗？

可以。NaiveProxy 使用 `443/tcp`，Hysteria2 使用 `443/udp`，同一个域名可以同时指向同一台服务器。

### Hysteria2 为什么有时会沿用 NaiveProxy 配置？

新版脚本会优先复用当前目录里 NaiveProxy `.env` 的配置：

- `HY2_DOMAIN` 默认沿用 `DOMAIN`
- `HY2_PASSWORD` 默认沿用 `PASSWORD`
- `HY2_MASQUERADE_URL` 默认沿用 `https://DECOY_DOMAIN`

这样两个协议可以更容易保持同域名、同密码、同伪装站。你也可以通过 `--hy2-domain`、`--hy2-password`、`--hy2-hop-iface`、`--hy2-masquerade` 显式覆盖。

### 为什么卸载 NaiveProxy 后 Caddy 服务还在？

当前脚本的 `--uninstall` 会移除 Caddyfile 里的 `forward_proxy` 配置，但默认保留 Caddy 服务、证书目录和伪装站点能力。这样做是为了避免影响正在复用 Caddy 证书的 Hysteria2。

如果只想清理项目目录里的 NaiveProxy `.env`，使用：

```bash
./deploy.sh --uninstall --purge
```

### 项目是什么开源协议？

PixelCat Proxy 当前使用 GNU General Public License v3.0。你可以自由使用、复制、修改和分发；如果分发修改版或衍生作品，需要继续按照 GPLv3 开源，并保留原版权声明和许可证文本。
