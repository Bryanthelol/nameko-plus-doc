---
#search:
#  boost: 2 
#search:
#  exclude: true
hide:
  - footer
---

# 概述

在分布式环境下，Nameko 提供了一种微服务架构方式，使得微服务间的通信变得简单，调用一个服务就像调用了一个本地方法一样。

Nameko 提供 RPC 同步调用（同时支持异步调用），事件发布/订阅，以及暴露 HTTP 协议的 API（但一般不建议使用，而是使用其他 web 框架）。

以上这一切的能力都基于实现 AMQP 协议的 RabbitMQ 之上达成。

当你启动一个 Nameko 微服务进程，服务的发现已经通过 RabbitMQ 完成，所以对于开发者来说，接下来只需要关注业务逻辑的实现即可。

如果业务量加大，对微服务的请求数量上升到了一定程度，那么只需要增加 Nameko 微服务的实例数量就可以轻松应对，底层的 RabbitMQ 会自动对 Nameko 实例做负载均衡。

这份文档接下来的内容会进一步说明：如何基于 Nameko 开发微服务应用。


## 参考文档

- nameko
  
    - https://nameko.readthedocs.io/en/v3.0.0-rc/
    - https://agitated-hoover-1d4d78.netlify.app/
    - https://hackernoon.com/building-microservices-with-nameko-part1-ud1135ug
    - https://morioh.com/p/64fd8227f38d
    - grpc

        - https://itnext.io/grpc-apis-with-nameko-11bbd9492225

- apiflask
    	- https://apiflask.com/
