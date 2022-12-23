### Kubernetes 基础概念

**1、Kubernetes 架构解析**

- master 节点
- node 节点

**2、Pod 的概念**

1) Pod 状态
- Pending: 挂起状态，已被系统接收，但有容器还未创建
- Running: 运行状态，Pod 在绑定的节点上正常运行
- Succeeded: 成功状态，所有容器执行成功并终止，并且不会再次重启
- Failed: 失败状态，所有容器都已终止，并且有一个容器以失败的方式终止
- Unknown: 未知状态，通常是由通信问题造成的
- ImagePullBackOff/ErrImagePull: 镜像拉取失败
- CrashLoopBackOff: 容器启动失败，可以通过 logs 命令查看具体的原因

2) Pod 探针
- Pod 的探针方式
  - ExecAction: 在容器内执行一个指定的命令，判断返回值
  - TCPSocketAction: 通过 TCP 链接检查容器的指定端口，如果端口开发，则认为容器健康
  - HTTPGetAction: 通过指定的 URL 进行 Get 请求，如果状态码在 200~400，则认为容器健康
- Pod 探针类型
  - startupProbe: 判断容器内的应用程序是否已经启动
  - livenessProbe: 探测容器是否运行
  - readinessProbe: 探测容器内的程序是否健康（程序是否就绪）

3) Pod 镜像拉取策略
- Always: 总是拉取
- Never: 无论是否存在都不拉取
- IfNotPreset: 镜像不存在时拉取镜像，默认，排除 latest

4) Pod 重启策略
- Always: 默认策略，容器失效时，自动重启该容器
- OnFailure: 容器不为 0 的状态码终止，自动重启该容器
- Never: 无论何种状态，都不会重启


