---
title: 节点体检清单：连接前先看这四个指标
date: 2026-04-26 10:20:00
tags:
  - 节点
  - 延迟
  - 排障
---

很多连接失败并不是客户端坏了，而是节点状态已经不适合当前网络。像素猫的习惯是先做体检，再改配置。

![节点体检清单示意图](/images/network-health-check.svg)

<div class="video-embed">
  <iframe src="https://www.youtube.com/embed/M7lc1UVf-VE" title="像素猫科学上网 ICU 视频展示" loading="lazy" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>

## 1. 延迟看趋势，不看单点

单次 ping 到 80ms 不代表稳定，连续十几次从 80ms 跳到 900ms 才是问题。优先记录最小值、最大值和抖动范围。如果抖动超过平均延迟的两倍，视频会议和远程桌面都会明显卡顿。

## 2. 丢包比高延迟更致命

轻微高延迟还可以靠缓冲解决，丢包会直接导致网页半开、图片缺块、SSH 卡死。连续测试 3 轮，每轮 30 秒，只要丢包稳定出现，就先换线路。

## 3. DNS 是否走对出口

打开代理后，DNS 仍然走本地运营商，可能导致域名被解析到错误区域。排查时可以分别测试直连和代理模式下的 DNS 结果，确认敏感域名不会被本地 DNS 抢答。

## 4. 目标站点可访问性

节点能连通不等于目标网站能打开。至少测试搜索、代码托管、文档站和视频站四类目标，避免只用一个站点判断线路质量。

## 快速结论

- 延迟平稳但偏高：可以用于阅读和下载。
- 延迟忽高忽低：不适合实时任务。
- 有规律丢包：优先换节点。
- DNS 出口异常：先检查客户端的 DNS 设置。
