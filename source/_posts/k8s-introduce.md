---
title: Kubernetes简介
date: 2021-01-05 15:33:02
categories: 系统架构
tags: Kubernetes
---

`Kubernetes是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了**应用部署，规划，更新，维护**的一种机制。`

<!--more-->


## 设计架构

![](https://feisky.gitbooks.io/kubernetes/architecture/images/14937095836427.jpg)

- 核心层：Kubernetes最核心的功能，对外提供API构建高层的应用，对内提供插件式应用执行环境
- 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS解析等）
- 管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy等）
- 接口层：kubectl命令行工具、客户端SDK以及集群联邦
- 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴
	1. 	Kubernetes外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS应用、ChatOps等
	1. 	Kubernetes内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等

## 集群架构

![](https://www.redhat.com/cms/managed-files/kubernetes_diagram-v3-770x717_0.svg)

Kubernetes集群包含有节点代理kubelet和Master组件(APIs, scheduler, etc)，一切都基于分布式的存储系统。我们把服务分为运行在工作节点上的服务和组成集群级别控制板的服务。Kubernetes节点有运行应用容器必备的服务，而这些都是受Master的控制。

**核心组件:
**

1. etcd保存了整个集群的状态；
1. apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
1. controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
1. scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
1. kubelet负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；
1. Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）；
1. kube-proxy负责为Service提供cluster内部的服务发现和负载均衡；

### Pod

Pod是Kubernetes中能够创建和部署的最小单元，是Kubernetes集群中的一个应用实例，总是部署在同一个节点Node上。Pod中包含了一个或多个容器，还包括了存储、网络等各个容器共享的资源。Pod支持多种容器环境，Docker则是最流行的容器环境。Pod中的容器共享相同的数据和网络地址空间，Pod之间也进行了统一的资源管理与分配。

Pod是K8s集群中所有业务类型的基础，可以看作运行在K8s集群中的小机器人，不同类型的业务就需要不同类型的小机器人去执行。目前K8s中的业务主要可以分为长期伺服型（long-running）、批处理型（batch）、节点后台支撑型（node-daemon）和有状态应用型（stateful application）；分别对应的小机器人控制器为Deployment、Job、DaemonSet和PetSet。

