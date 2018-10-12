---
title: Istio-Production-Case
date: 2018-10-10 09:28:41
tags:
  - Istio
---

# Istio 生产案例

## 美国房产交易平台——Trulia

[Microservice Observability with Istio](https://www.trulia.com/blog/tech/microservice-observability-with-istio/)

[Istio和Kubernetes帮助Trulia房产网站消除单体架构增强微服务的可观测性](http://www.servicemesher.com/blog/microservice-observability-with-istio/)

## HR SaaS(人力资源系统)——Namely

[Livestream : Services at Namely (Istio, Spinnaker, +) and some Machine Learning (Kubeflow)](https://www.youtube.com/watch?v=iga5peu4E88)

[Using Istio for developing locally - Robert Ross (Namely)](https://www.safaribooksonline.com/videos/oscon-2018/9781492026075/9781492026075-video321571)

## Auto Trader

[Upgrading Istio](https://github.com/istio/istio/issues/3720)

## HP FitStation

[How HP is building its next-generation footwear personalization platform on Istio](https://istio.io/blog/2018/hp/)

## PubNub

[Istio 1.0: Come for Traffic Routing, Stay for Distributed Tracing](https://www.pubnub.com/company/news-coverage/2018/istio-1-0-come-for-traffic-routing-stay-for-distributed-tracing/)

## 蚂蚁金服、阿里大文娱UC事业部——SOFA Mesh

SOFAMesh 是基于 Istio 改进和扩展而来的 Service Mesh 大规模落地实践方案。在继承 Istio 强大功能和丰富特性的基础上，为满足大规模部署下的性能要求以及应对落地实践中的实际情况，有如下改进：

- 采用 Golang 编写的 MOSN 取代 Envoy
- 合并 Mixer 到数据平面以解决性能瓶颈(部分mixer功能)
- 增强 Pilot 以实现更灵活的服务发现机制
- 增加对 SOFA RPC、Dubbo 的支持

<!--more-->

### Golang版Sidecar——MOSN

![golang sidecar - MOSN](/images/istio-production-case/golang-sidecar.png)

Golang版本Sidecar参考了Envoy，非常明确的实现XDS API。因为XDS API是目前的事实标准，实现XDS API使其让兼容Istio。

在协议支持上，MOSN会支持标准的HTTP/1.1和HTTP/2。然后会增加一些特殊的协议扩展，包括 SOFA协议，Dubbo协议，HSF协议。

### Mixer部分功能被合并至数据平面

![mixer merge](/images/istio-production-case/mixer-merge.png)

最大的变化在Mixer，其实刚才的goland版 sidecar虽然是全新编写，但是说白了是对Envoy的替换，在架构上没有什么变化。但这一步的变化就非常大，我们会合并一部分的Mixer功能。

Mixer的三大功能：

- check。也叫precondition，前置条件检查，比如说黑白名单，权限。
- quota。比如说访问次数之类。
- report。比如说日志，度量等。

三大功能里面，前两个功能是同步阻塞的，就是一定要检查通过，或者是说quota验证OK，才能往下走。如果结果没回来只能等，因为这是业务逻辑，必须要等。而Report是可以通过异步和批量的方式来做的。

此处，SOFA Mesh给出的合并方案是：将mixer原来的两个部分 check 和 quota 合并进来，原有 report 部分继续保留在mixer里面。

### 增强版Pilot

![enhanced pilot](/images/istio-production-case/pilot-enhance.png)

SOFA Mesh大幅扩展和增强Istio中的Pilot模块:

- 增加SOFARegistry（蚂蚁金服内部的服务注册中心）的Adapter，提供超大规模（服务数以万计）服务注册和发现的解决方案
- 增加数据同步模块，以实现多个服务注册中心之间的数据交换
- 增加Open Service Registry API，提供标准化的服务注册功能，因为现在Istio的方案只有服务发现，它的服务注册是走k8s的，用的是k8s的自动服务注册。如果想脱离k8s环境，就要提供服务注册的方案。

### Roadmap

- 8月底发布 0.2.0 版本
- 9月份启动在 UC 的落地
- 蚂蚁主站落地
