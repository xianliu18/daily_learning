### Docker-Compose 部署 Hadoop 集群
#### 1，简化版步骤：
1，创建目录：`D:\big_data`，准备 [`hadoop-3.2.3.tar.gz`](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/) 和 [`jdk-8u192-linux-i586.tar.gz`](https://repo.huaweicloud.com/java/jdk/)
2，准备 `Dockerfile` 文件；
3，构建镜像：`docker build -t hadoop .`；
4，创建 `docker-compose.yml`；
5，启动 docker-compose，`docker-compose up -d`；
6，进入 hadoop101 容器内部：`docker exec -it hadoop101 /bin/bash`；
7，免密登录处理；
8，编写同步脚本；

#### 2，详细步骤：
1，创建目录：`D:\big_data`，当前为空目录，完成以下步骤后的目录，如下图所示：

![最终目录结构](./images/最终目录.png)

2，准备 `Dockerfile` 配置文件，内容如下：

<details>
<summary>Dockerfile 详细内容</summary>

```sh
FROM centos:centos7.9.2009
MAINTAINER noodles

# 切换工作目录
WORKDIR /opt/hdp

# 复制 hadoop java 文件
ADD hadoop-3.2.3.tar.gz ./
ADD jdk-8u192-linux-i586.tar.gz ./

# 安装 ssh 服务
RUN yum install -y openssh-server openssh-clients rsync glibc.i686 vim\
        && ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N "" -q \
        && ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N "" -q \
        && ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N "" -q \
        && groupadd hadoop \
        && useradd hdfs -g hadoop \
        && echo "root:root" | chpasswd \
        && echo "hdfs:hdfs" | chpasswd \
        && mkdir /var/run/sshd \
        && mkdir /opt/hdp/tmp \
        && chown -R hdfs:hadoop /opt/hdp \
        && mv hadoop-3.2.3 hadoop \
        && mv jdk1.8.0_192 jdk

# 暴露端口
EXPOSE 22 9870 8088 50070

# 预定持久化文件的目录
VOLUME /opt/hdp/tmp

# 运行 ssh 服务
CMD ["/usr/sbin/sshd","-D"]
```
</details>

3，构建镜像：

```sh
docker build -t hadoop .
```

4，准备 `docker-compose.yml` 文件

<details>
<summary>docker-compose.yml 详细内容</summary>

```yaml
version: "3.0"

services:
  hadoop-master:
    image: hadoop:latest
    ports:
      - "9870:9870"
      - "50070:50070"
      - "8088:8088"
    networks:
      - docker0
    volumes:
      - v_master:/opt/hdp/tmp
    container_name: hadoop101

  hadoop-slave1:
    image: hadoop:latest
    networks:
      - docker0
    volumes:
      - v_slave1:/opt/hdp/tmp
    container_name: hadoop102

  hadoop-slave2:
    image: hadoop:latest
    networks:
      - docker0
    volumes:
      - v_slave2:/opt/hdp/tmp
    container_name: hadoop103

networks:
  docker0:

volumes:
  v_master:
  v_slave1:
  v_slave2:
```
</details>

5，后台启动 docker-compose，`docker-compose up -d`；
6，进入 hadoop101 容器内部：`docker exec -it hadoop101 /bin/bash`；
7，配置免密登录，**每个服务器**均需要执行：

```sh
# 先切换用户
su hdfs
# 免密登录
ssh-keygen -t rsa
ssh-copy-id hadoop101
ssh-copy-id hadoop102
ssh-copy-id hadoop103
```

8，同步脚本：`/opt/hdp/shell/xsync.sh`

<details>
<summary>xsync.sh 详细内容</summary>

```sh
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if [ $pcount == 0 ]
then
  echo no args;
  exit;
fi

#2 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4 获取当前用户名称
user=`whoami`

#5 循环
for host in hadoop101 hadoop102 hadoop103
do
        echo ------------------- $host --------------
        rsync -rvl $pdir/$fname $user@$host:$pdir
done
```
</details>

9，`~/.bashrc` 中配置 JAVA_HOME，HADOOP_HOME 全局变量：

```sh
export JAVA_HOME=/opt/hdp/jdk
export HADOOP_HOME=/opt/hdp/hadoop
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

让环境变量生效，`source ~/.bashrc`；

10，配置 Hadoop

<details>
<summary>hadoop-env.sh</summary>

```xml
JAVA_HOME=/opt/hdp/jdk
```
</details>

<details>
<summary>core-site.xml 配置</summary>

```xml
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hadoop101:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/opt/hdp/tmp</value>
                <description>Abase for other temporary directories.</description>
        </property>
        <property>
                <name>hadoop.http.staticuser.user</name>
                <value>hdfs</value>
        </property>
</configuration>
```
</details>

<details>
<summary>hdfs-site.xml 配置</summary>

```xml
<configuration>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>hadoop102:50090</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/opt/hdp/tmp/dfs/nn</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/opt/hdp/tmp/dfs/dn</value>
        </property>
</configuration>
```
</details>

<details>
<summary>mapred-site.xml 配置</summary>

```xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>hadoop103:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>hadoop103:19888</value>
        </property>
</configuration>
```
</details>

<details>
<summary>yarn-site.xml 配置</summary>

```xml
<configuration>
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>hadoop103</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
</configuration>
```
</details>

<details>
<summary>workers 配置</summary>

```xml
hadoop101
hadoop102
hadoop103
```
</details>

10.1 同步配置文件，`shell/xsync hadoop/`

10.2 格式化 NameNode
```sh
hdfs namenode -format
```

10.3 启动集群
```sh
start-dfs.sh
start-yarn.sh
```

10.4 访问集群：http://localhost:9870

<br/>
- 参考资料：
  - [Docker 部署 Hadoop 集群](https://kpretty.tech/archives/docker2)