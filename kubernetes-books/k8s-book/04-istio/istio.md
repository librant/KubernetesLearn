
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
- 安全认证 （Citaldel）

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
- Citaldel 服务安全的主要组件，用于秘钥和证书的管理

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