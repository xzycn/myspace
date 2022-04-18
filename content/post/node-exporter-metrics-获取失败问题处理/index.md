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
  filename: featured
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
error encoding and sending metric family: write tcp 192.168.1.2:9100" msg="->x.x.x.x:41412: write: broken pipe"
```