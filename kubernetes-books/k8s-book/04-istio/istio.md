
#### 深入理解 Istio 云原生服务网格进阶实战

**目录：**

第一章：ServiceMesh 概述
第二章：核心功能   
第三章：架构解析   
第四章：安装与部署    
第五章：流量控制   
第六章：可观察性   
第七章：安全    
第八章：进阶实战    
第九章：故障排查    
第十章：ServiceMesh 生态

---
**第一章：ServiceMesh 概述**

1) 基本概念   
- 本质：基础设施层
- 功能：请求分发
- 部署形式：网络代理
- 特点：透明

2) 总体架构
- 数据平面（Data Plane）
  - 按照无状态目标设计的
  - 直接处理入站和出站数据包（转发，路由，健康检查，负载均衡，认证，鉴权，监控数据等）
  - 对用户无感知
- 控制平面（Control Plane）
  - 不直接解析数据包
  - 与控制平面中的代理通信、下发策略和配置
  - 负责网络行为的可视化
  - 通常提供 API 或命令行工具

服务网格的出现带来了哪些变革：
- 第一：服务治理与业务逻辑解耦
- 第二：异构系统的统一治理

服务网格的优势：
- 可观测性   
- 流量控制
- 安全

服务网格的局限：
- 增加了复杂度
- 运维人员需要更专业
- 延迟
- 平台的适配

kubernetes 与 ServiceMesh 服务访问关系：   
- 流量转发
- 服务发现

xDS 协议：   
- CDS/EDS/LDS/RDS （v2）
- SRDS/VHDS/SDS/RTDS (v3)

Envoy 基本术语：
- Downstream：下游
- Upstream：上游
- Lister：监听器
- Cluster：集群

Istio 主要功能： 
- 流量管理
- 策略控制
- 可观测性（kiali）
- 安全认证 （Citadel）

Istio 中定义的 CRD：
- Gateway：描述了网络边缘运行的负载均衡器
- VirtualService：实际上可以将 kubernetes 服务连接到 Istio Gateway 上，并且可以执行更多的操作
- DestinationRule：决定了经过路由处理之后的流量访问策略
- EnvoyFilter：描述了针对代理服务的过滤器
- ServiceEntry：在默认情况下， Istio 服务网格中的服务服务发现 Mesh 以外的服务，
  ServiceEntry 能够在 Istio 内部的服务注册表中加入额外的条目

---
**第二章：核心功能**

1) 流量控制    
- 请求路由和流量转移
- 弹性功能（熔断，超时和重试）
- 调试能力（故障注入和流量镜像）

2) 实现流量控制的自定义资源
- VirtualService：用于网格内的路由设置
- DestinationRule：定义路由的目标服务和流量策略
- ServiceEntry：注册外部服务到网格内，并对其流量进行管理
- Gateway：用来控制进出网格的流量，包括入口、出口网关
- Sidecar：对 Sidecar 代理进行整体设置
- WorkloadEntry/WorkloadGroup：将虚拟机接入网格

3) 安全   
- Citadel 服务安全的主要组件，用于秘钥和证书的管理

1、认证    
- 对等认证（Peer Authentication）：用于服务到服务的认证
- 请求认证（Request Authentication）：最终用户认证

2、授权   
- 不同级别的访问控制（网格级别，命名空间级别，工作负载级别）
- AuthenticationPolicy：自定义 CRD，定义策略指定的目标和动作

4) 可观察性   
- 及时反馈异常或风险
- 在出现问题时，能够快速定位问题根源并解决问题
- 收集并分析数据

istio 提供三种不同类型的数据：
- 指标（Metrics）
- 日志（Access Logs）
- 分布式追踪（DistributionTraces）

---
**第三章：架构解析**

##### 1) 控制平面    

1、Pilot
- 管理和配置部署在特定 Istio 服务网格中所有 Sidecar 代理实例

**Pilot 架构：**   
- 抽象模型（Abstract Model）
  - 实现对不同服务注册中心的支持
  - 关键成员
    - HostName（service 名称）
    - Ports（service 端口）
    - Address（service ClusterIP）
    - Resolution（负载均衡策略）
- 平台适配器（Platform Adapters）
  - Pilot 基于平台适配器，实现服务注册中心数据和抽象模型数据
- xDS API
  - Pilot 使用了一套源于 Envoy 项目的标准数据平面 API，将服务信息和流量规则下发到数据平面的 Sidecar 中
    - LDS：Listener 发现服务，控制 Sidecar 启动端口监听
    - RDS：Router 发现服务，用于 HTTP 连接管理过滤器动态获取路由配置
    - CDS：Cluster 发现服务，用于动态获取 Cluster 信息
    - EDS：Endpoint 发现服务，用于动态维护端点信息
- 用于 API
  - Pilot 定义了一套用户 API，用户 API 提供了面向业务的高层抽象，可以被运维人员理解和使用

** Pilot 实现：**
[Pilot] Discovery Service --> xDS --> Envoy(agent)
- Discovery Service：pilot-discovery，从 Service Provider 中获取服务信息
  - 服务注册中心，Istio 控制平面与 Sidecar 之间的桥梁角色
    - 监控服务注册中心的服务注册情况
    - 监控 Istio 控制平面的信息变化
      - RouterRule
      - DestinationRule
      - VirtualService 
      - Gateway 
      - EgressRule 
      - ServiceEntry 
  - 初始化和启动
    - 创建 kubernetes client（initKubeClient）
    - 读取 Mesh 配置（initMeshConfiguration）
    - 连接初始化与配置存储中心（initConfigController）
    - 连接配置与服务注册中心（initServiceController）
    - 初始化 Discovery 服务（initDiscoveryService）
    - 启动 gRPC Server 并接收来自 Envoy 端的连接请求
    - 接收 Sidecar 端的 xDS 请求 --> (ConfigController/ServiceController) 获取配置 --> Sidecar
- agent：pilot-agent，根据 kubernetes API Server 中的配置信息生成 Envoy 的配置文件，负责启动，监控 Sidecar 进程
  - 生成 Sidecar 配置
  - Sidecar 的启动与监控
    - 创建 Envoy 对象（proxyConfig, role.serviceNode, loglevel, pilot SA）
    - 创建 agent 对象（epochs map, channel, statusCh）
    - 创建 watcher， 包含 证书和 agent.Restart()
    - watcher.Run() --> agent.Restart()
    - 创建 context，用于协程信号进行通知
    - agent.Run() --> 监听 statusCh
- Proxy：Sidecar Proxy，所有服务的流量代理，直接连接 pilot-discovery，间接地从 kubernetes 等服务注册中心获取集群中微服务的注册情况
- Service A/B：使用了 Istio 的应用，进出网络流量会被 Proxy 接管

2、Citadel
- 负责身份认证和证书管理的核心安全组件（istiod）

**Citadel 基本功能：**
- 证书签发机构（CA）负责秘钥和证书管理
- API 服务器将安全配置分发给数据平面
- 客户端和服务端通过代理进行安全通信
- Envoy 代理管理遥测和审计

**Citadel 工作原理：**
- CA 服务器
  - CA 签发机构是一个 gRPC 服务器
    - CA 服务：用来处理 CSR 请求
    - 证书服务：签发证书
  - CreateCertificate()：证书签发
- SDS 服务器
  - Secret Discovery Service，在运行时动态获取证书和私钥的 API
  - SDS 优点：
    - 无需挂载 Secret 卷
    - 动态更新证书，无需重启
    - 可以监听多个证书秘钥对
- 证书秘钥控制器
  - 证书秘钥控制器可以监听 istio.io/key-and-cert 类型的 Secret 资源
  - 周期检查证书是否过期，并更新证书
- 证书轮换
  - Rotator 自动检查自签名的根证书，并在证书即将过期时自动更新
  - 获取当前的证书，解析证书的有效期并获取下一次轮换时间
  - 启动定时器，如果发现证书到达轮换时间，则从 CA 中获取最新的证书秘钥对
  - 更新证书

SDS 工作流程：
- Envoy 通过 SDS API 发送证书和秘钥请求
- istio-agent 作为 Envoy 代理，创建一个私钥和证书请求 CSR，并发送给 istiod
- 证书签发机构验证 CSR 并生成证书
- istio-agent 将私钥和从 istiod 中收到的证书通过 SDS API 发送给 Envoy 
- 周期性执行，实现秘钥和证书的轮换

3、Galley
- 提供配置验证功能，还负责配置的管理和分发

**配置验证功能：**
- 使用 kubernetes 提供的一个 Admission Webhook -- ValidatingWebhookConfiguration 来做配置的验证
- istio-galley
  - /admitmixer --> wh.serveAdmitMixer
  - /admitpilot --> wh.serveAdmitPilot

**配置管理实现：**
- Processor 
  - Source：代表 Galley 管理配置来源
  - Handler：对 "配置" 事件进行处理的处理器
  - State：Galley 管理的配置在内存中状态
- MCP（gRPC Server）

**MCP 协议：**
MCP 提供了一套用于配置订阅和分发的 API 
- source：配置提供端，在 Istio 中， Galley 即 source 
- sink：配置的消费端，在 Istio 中，包括 Pilot/Mixer 组件
- resource：source 和 sink 关注的资源体，就是 Istio 中的 "配置"

提供两个 service：
- ResourceSource
- ResourceSink


##### 2) 数据平面 

1、数据平面主要负责执行任务
- 服务发现：探测所有可用的上游或后端服务实例
- 健康检测：探测上游或后端服务实例是否健康，是否准备好接收网络流量
- 流量路由：将网络请求路由到正确的上游或后端服务
- 负载均衡：在对上游或后端服务请求时，选择合适的服务实例接收请求，同时负责处理超时，短路，重试等情况
- 身份认证和授权：对网络请求进行身份认证，权限认证
- 链路追踪：对每个请求生成详细的统计信息

2、常见数据平面实现
- Envoy
- MOSN 
- Linkerd 






