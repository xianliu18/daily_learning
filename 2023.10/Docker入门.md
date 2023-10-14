### Docker 入门

![Docker 架构图](https://docs.docker.com/get-started/images/docker-architecture.png)

#### 1，Docker 安装

#### 2，Docker 常用命令

#### 3，Docker 数据挂载

<br>
<hr>
<br>

### K8s 相关
#### 1，安装 K8s
- kubelet
- kubectl
- kubeadm

#### 2，kubectl 常用命令


#### 3，K8s 工作负载
- Deployment：无状态应用部署，比如微服务，多副本等功能；
- StatefulSet：有状态应用部署，比如 Redis，提供稳定的存储、网格等功能；
- DaemonSet：守护型应用部署，比如日志收集组件，在每个机器都运行一份；
- Job/CronJob：定时任务部署，比如垃圾清理组件；

#### 4，服务网格 Service

#### 5，Ingres
- Service 的统一网关入口

#### 6，存储抽象
- 以 NFS 为例

#### 7，PV & PVC
- PV(Persistent Volume)：将应用需要持久化数据保存到指定位置；
- PVC(Persistent Volume Claim):申明需要使用的持久卷规格，即容量大小；

#### 8，ConfigMap
- 挂载配置文件

#### 9，Secret
- 保存密码、OAuth 令牌和 SSH 密钥；

#### 10，DevOps
- CI(continus integration)：持续集成
- CD(continus delivery/deployment)：持续交付


<br/>

- 参考资料：
- [docker 使用理解全流程](https://blog.csdn.net/qq_44787816/article/details/118103587)
