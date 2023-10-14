### Docker 入门

![Docker 架构图](https://docs.docker.com/get-started/images/docker-architecture.png)

#### 1，Docker 安装

#### 2，Docker 常用命令
#### 镜像相关：
- 列出本地主机上的镜像：`docker images`
- 拉取指定版本的镜像：`docker pull nginx:1.20.1`
- 移除指定版本的镜像：`docker rmi nginx:1.20.1`

#### 容器相关：
- 列出所有正在运行的容器：`docker ps`
- 列出所有的容器，包括未运行的：`docker ps -a`
- 启动容器，并映射 88 端口：`docker run --name=mynginx -d --restart=always -p 88:80 nginx`
	- `-d`：后台运行；
	- `88:80`：88 主机中的端口，80 容器中的端口；
- 进入到容器内部修改：`docker exec -it 容器ID /bin/bash`
- 挂载数据到外部修改：
  - `docker run --name=mynginx -d --restart=always -p 88:80 -v /data/html:/usr/share/nginx/html:ro nginx`
  - `/data/html`：主机中的地址；
  - `/usr/share/nginx/html`：容器内地址；
  - `ro`：只能在主机上修改；

<br>
<hr>
<br>

### K8s 相关
#### 1，安装 K8s
- kubelet
- kubectl
- kubeadm
- K8s 集群：
  - N Master Node(主节点) + N Worker Node(工作节点)

#### 2，kubectl 常用命令
- 查看集群所有节点：`kubectl get nodes`
- 根据配置文件，给集群创建资源：`kubectl apply -f XXX.yaml`
- 查看集群部署了哪些应用：`kubectl get pods -A`
- 创建 Pod：`kubectl run mynginx --image=nginx`
  - `mynginx`：pod 的名称，默认创建在 default 名称空间中；
- 删除 Pod：`kubectl delete pod mynginx -n XXXX`
  - `-n XXX`：指定某个名称空间；

#### 2.1 名称空间
- 查看指定名称空间：`kubectl get pod -n kubenetes-dashboard`
- 创建名称空间：`kubectl create ns hello`
- 删除名称空间：`kubectl delete ns hello`


#### 3，K8s 工作负载
- Deployment：无状态应用部署，比如微服务，多副本等功能；
- StatefulSet：有状态应用部署，比如 Redis，提供稳定的存储、网格等功能；
- DaemonSet：守护型应用部署，比如日志收集组件，在每个机器都运行一份；
- Job/CronJob：定时任务部署，比如垃圾清理组件；

#### 3.1 Deployment 部署
- Deployment 部署：`kubectl create deployment mytomcat --image=tomcate:8.5.68`
- 查看部署的内容：`kubectl get deploy`
- 删除部署：`kubectl delete deploy mytomcat`
- 查看 Pod 部署在哪台机器：
  - `kubectl get pod -o wide`
  - `kubectl get pod -w`
- 多副本
- 扩缩容
  - 扩容：`kubectl scale deploy/my-dep --replicas=5`
  - 缩容：`kubectl scale deploy/my-dep --replicas=2`
- 自愈

- [常见命令](https://kubernetes.io/zh-cn/docs/reference/kubectl/)
- [命令详细解释](http://docs.kubernetes.org.cn/626.html)

#### 4，服务网格 Service
- 将资源暴露为新的 kubernetes service：
  - `kubectl expose deployment my-dep --port=80 --target-port=8000`
  - 将 deployment 中 my-dep 通过 service 的 80 端口转发至容器的 8000 端口；
- 资源包括：pod(po), service(svc), replication controller(rc), deployment(deploy), replica set(rs)


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
