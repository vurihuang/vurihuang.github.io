---
layout: post
title: 【译】在 Kubernetes 中关于长连接的负载均衡和扩容的介绍
slug: kubernetes-long-lived-connections
---

---
{: data-content="原文" }

[Load balancing and scaling long-lived connections in Kubernetes](https://learnk8s.io/kubernetes-long-lived-connections)

> TL;DR Kubernetes 不支持针对长连接的负载均衡，有些 Pods 相比可能会接收到更多的请求。如果你使用的是 HTTP/2, gRPC, RSockets, AMQP 或者其他类似数据库连接这种长连接方式，那么你可能需要考虑使用客户端层面的负载均衡。

