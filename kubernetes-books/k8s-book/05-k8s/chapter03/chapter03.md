### Docker 基础

**1、Docker 介绍**

Docker 利用 Linux 资源分离机制（Cgroup 和 Namespace）来闯将相互独立的容器（Container）

Docker 在逻辑上分成一下几个部分：
- Docker client: Docker 的客户端，用于执行 Docker 的相关命令，比如镜像下载
- Docker Daemon: Docker 的守护进程
- Docker Image: Docker 镜像，用于启动容器实例
- Docker Conatainer: Docker 容器，由镜像启动，容器内运行相关的应用程序

**2、Docker 基本命令**

1) docker version: 查看 Docker 版本
- OCI：Open Container Initiative 的简称，容器格式和 Runtime 的标准
- Containerd: Docker 为了兼容 OCI 标准，将容器 Runtime 及其管理功能从 Docker 守护进程中剥离出来，
  用于不启动 Docker 也能通过 Containerd 来管理容器
- RunC: Docker 按照 OCF(Open Container Format) 制定的一个轻量级工具，可以使用 RunC 不通过 Docker 引擎即可
  实现容器启动，停止和资源隔离等功能

```shell
# 查看 Docker 详细信息
docker info
# 查询指定镜像
docker search image_name
# 拉取指定镜像
docker pull image_name

# 指定镜像运行容器
docker run -itd image_name bash
docker run -it -d image_name -p 1111:80 -v /etc/hosts:/etc/hosts bash

# 登录正在运行的容器
docker exec -it containerID bash

# 查看当前运行的容器
docker ps
docker ps -aq

# 文件复制
# 将主机文件拷贝到容器中
docker cp ./hostfile.md containerID:/tmp
# 将容器中的文件拷贝到主机
docker cp containerID:/tmp/temp.md ./hostdir

# 查看容器运行日志
docker logs -f containerID/container_name --tail 1

# 查看容器详细信息
docker inspect containerID

# 构建容器镜像
docker build -t image_name:image_tag .
```

**2、Dockerfile 编写**

1) Dockerfile 常用命令
- FROM: 继承基础镜像
- MAINTAINER: 镜像制作者的信息
- LABEL: k=v 的形式，定义镜像相关的元数据
- RUN: 用来执行 shell 命令
- EXPOSE: 暴露端口号
- CMD: 启动容器默认执行的命令，会被覆盖
- ENTRYPOINT: 启动容器真正执行的命令，不会被覆盖
- VOLUME: 创建挂载点
- ENV: 配置环境变量
- ADD: 复制文件到容器，一般复制文件，压缩包自动解压
- COPY: 复制文件到容器，一般复制目录
- WORKDIR: 设置容器的工作目录
- USER: 容器使用的用户
- ARG: 设置编译镜像时传入的参数

2) Dockerfile 实例

- 下载 nginx 安装包
```shell
wget -c https://nginx.org/download/nginx-1.12.2.tar.gz
```

- 编写 Dockerfile 文件
```shell
# 参考 ExampleDockerfile
```

- 执行编译命令
```shell
docker build -t librant_test_nginx:1.0 .
```

- 测试生成的镜像
```shell
docker run -d -p 8080:80 --name=librant_nginx librant_test_nginx:1.0
```
