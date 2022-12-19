
#### Traffic Management

**1、流量控制 CRD** 
- VirtualService
  - 控制流量转发规则及 API 粒度治理功能，包括错误注入，域控制等
- DestinationRule
  - 抽象路由转发的上游目标服务及其相关配置项
- ServiceEntry
  - 将服务网格之外的 IP 流量注册到 Istio 中
- Gateway
  - 一般用于 Ingress 和 Egress, 定义服务网格的初入口及相关治理规则
- Sidecar
  - 用于声明服务网格中服务间的依赖关系

1) VirtualService
- 在 VirtualService 中定义了一组路由规则, 当流量进入的时，则逐个规则进行匹配，直到匹配成功后将流量转发给指定的路由地址

关键字段解析：
- hosts: 用来配置下游访问的可寻址地址
- match: 用来配置路由规则（uri/method/authority/headers/port/queryParam）
- route: 用来配置路由转发目标规则，可以指定需要访问的 subset(服务子集)

2) DestinationRule
- 定义网格中某个 service 对外提供的服务策略及规则，包括负载均衡策略、异常点检测、熔断控制、访问连接池等

负载均衡策略：
- 简单的负载均衡策略
  - ROUNDROBIN：轮询
  - LEASTCONN: 最小链接数
  - RANDOM：随机
  - PASSTHROUGH
- 一致性 Hash 策略
- 区域性负载均衡策略

3) Gateway 
- 定义了所有 HTTP/TCP 流量进入网格的统一入口和从网格中出站的统一出口

4) ServiceEntry
- 将网格外的服务注册到 Istio 的注册表中，可以把外部服务当做和网格内部的服务一样进行管理和操作

global.outbandTrafficPolicy.mode:
- ALLOW_ANY: Istio 代理允许调用未知服务
- REGISTRY_ONLY: Istio 代理会阻止任何没有在网格中定义的 HTTP 服务或 ServiceEntry 的主机

5) Sidecar
- 在默认情况下，Istio 中所有 Pod 的 Envoy 代理都是可以被寻址的

---







