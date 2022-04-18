---
title: Node Exporter metrics 获取失败问题处理
date: 2022-04-18T07:43:18.740Z
draft: false
featured: false
authors:
  - Lordran
tags:
  - Prometheus
  - NodeExporter
  - OPS
  - 运维
categories:
  - 监控
image:
  filename: nodeexporter.png
  focal_point: Smart
  preview_only: false
---
在 Grafana 中查看某一 job 中 target 的列表中缺少了一些节点，查看作为数据源的 Prometheus 的日志，

```shell
> scrape_pool=portal-proxy-server target=http://node1:9100/metrics msg="Scrape failed" err="context deadline exceeded"
> scrape_pool=portal-proxy-server target=http://node2:9100/metrics msg="Scrape failed" err="context deadline exceeded"
```

接着查看对应节点上的 node exporter 日志：

```shell
error encoding and sending metric family: write tcp 192.168.66.74:9100" msg="->x.x.x.x:41412: write: broken pipe"
```

配合抓包：

![]()

判断是 Prometheus 断开了连接，查看了网络没发现问题，但是使用curl访问节点上的 metrics 时，用时竟然达到了10s，这大大超过了我们设置的scrape_timeout(2s)。查看 metrics 返回的内容，发现了很多 ipvs 相关的指标。