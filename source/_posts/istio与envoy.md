---
title: istio与envoy
categories:
  - istio
  - envoy
  - 云原生
  - kubernetes
date: 2021-11-04 14:51:58
tags:
---


### istio与envoy的关系

服务网格中的概念分为控制平面和数据平面，控制平面是istio，数据平面是envoy。

![service-mesh](https://istio.io/latest/img/service-mesh.svg)

> 图片摘自 istio.io

控制平面的作用可以理解为配置中心server(或者说管理server)，负责配置的获取和配置的下发。数据平面就是服务侧的透明代理，所有服务的流量都通过数据平面进行转发。

istio作为以k8s为基础的服务网格，配置项的设置是通过CRD(CustomResourceDefine)实现，配置下发到envoy是通过xDS协议，envoy通过gRPC stream订阅istio中的配置。

流程图如下:

![Multiple EDS requests on the same stream](https://www.envoyproxy.io/docs/envoy/latest/_images/eds-same-stream.svg)

注意是envoy去istio发现配置，而不是istio将配置发送给envoy，使用gRPC是为了更低延迟的更新配置

> xds协议的介绍，图片摘自envoy官方文档: https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol#xds-protocol 

istio也有支持其他数据平面的趋势，需要数据平面支持xds协议，但是目前看来istio还是离不开envoy，就像k8s目前离不开docker一样，两者的关系也有一定的相似。

### istio的优势

个人感觉istio最大的优势是**简化了envoy的配置**，原始的envoy配置是十分复杂的，有一定的使用成本，istio在这方面做了抽象，减少了使用者的使用成本。

比如: authorizationpolicies.security.istio.io这个CRD通过结合envoy的http-filter的ext_authz和rbac filter实现了对服务的自定义认证，简化了其中大量的配置，大大的减少了使用成本.

优先推荐使用gRPC的方式实现外部认证server，istio对于http的外部认证server的一些配置支持的比较慢，当想要使用http server实现方式时，有些配置在envoy中有，但是istio中没有，可能不得不去了解envoy的文档，自己写略微复杂一点的envoy filter，当然其中的调试工作也是比较复杂的。

> https://github.com/istio/istio/issues/27790#event-5281877115

