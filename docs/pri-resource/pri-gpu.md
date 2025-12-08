---
sidebar_position: 2
---

# GPU调度
**功能概述** 提供高可用的负载均衡服务，自动分发流量到后端服务器。

| 字段 | 用途 | 默认值 |                         备注                          | 说明 |
|:---:|:----:|:----:|:---------------------------------------------------:|:--:|
| name | 资源名（统一用于 deployment、svc、hpa、ingress） | mgtvxxx |                       mgtv-ai                       |  建议保持统一  |
| namespace | 所在命名空间 | default | [hub.imgo.tv/rc/rc_new_topic/rc_new_topic:v0.7.3](hub.imgo.tv/rc/rc_new_topic/rc_new_topic:v0.7.3) |  注意命名规范  |
| image | 容器镜像 | 无 |                          无                          |    |