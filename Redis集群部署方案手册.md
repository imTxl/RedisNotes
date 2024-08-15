# Redis集群部署方案手册

## 准备工作

> 1.redis-7.0.10.tar.gz
>
> 2.操作系统x86_64
>
> 3.SUSE及CentOS

## 1.创建用户和配置环境参数

### 1.1 创建用户和创建所需目录(如已存在就不用执行了)

```shell
groupadd redis
useradd -d /home/redis -g redis -m redis
chmod 755 /home/redis
mkdir -p /home/redis/software
mkdir -p /home/redis/yunwei
chown -R redis:redis /home/redis
mkdir -p /data/redis-cluster
chown -R redis:redis /data/redis-cluster
```

### 1.2 下载

> https://github.com/redis/redis/archive/7.0.10.tar.gz



### 1.3 调整配置文件

```shell
# 编辑 /etc/sysctl.conf
vi /etc/sysctl.conf 
# 新增如下：
vm.overcommit_memory = 1
net.core.somaxconn = 1024
---------------------------
# 刷新参数
sysctl -p
---------------------------
# 编辑 /etc/security/limits.conf
vi /etc/security/limits.conf
# 新增如下：
redis  soft    nofile          10032
redis  hard    nofile          10032
redis  soft    nproc           65535
redis  hard    nproc           65535
```



## 2.安装部署redis

### 2.1 部署python3.x环境

（略）

### 2.2 解压安装包

```shell
su - redis
tar zxf $HOME/software/redis-7.0.10.tar.gz -C $HOME/software
cd $HOME/software/redis-7.0.10/
make
make install PREFIX=$HOME/redis-7.0.10
```



### 2.3 创建所需目录并拷贝配置文件

```shell
# node A节点
mkdir -p $HOME/redis-7.0.10/redis-cluster/{7001,8001}
mkdir -p /data/redis-cluster/{7001,8001}
mkdir -p /data/redis-cluster/7001/{logs,data}
mkdir -p /data/redis-cluster/8001/{logs,data}
touch /data/redis-cluster/7001/logs/redis_7001.log
touch /data/redis-cluster/8001/logs/redis_8001.log
mkdir $HOME/redis-7.0.10/redis-cluster/conf
cp $HOME/software/redis-7.0.10/redis.conf $HOME/redis-7.0.10/redis-cluster/7001
cp $HOME/software/redis-7.0.10/redis.conf $HOME/redis-7.0.10/redis-cluster/8001
---------------------------
# node B节点
mkdir -p $HOME/redis-7.0.10/redis-cluster/{7002,8002}
mkdir -p /data/redis-cluster/{7002,8002}
mkdir -p /data/redis-cluster/7002/{logs,data}
mkdir -p /data/redis-cluster/8002/{logs,data}
touch /data/redis-cluster/7002/logs/redis_7003.log
touch /data/redis-cluster/8002/logs/redis_8003.log
mkdir $HOME/redis-7.0.10/redis-cluster/conf
cp $HOME/software/redis-7.0.10/redis.conf $HOME/redis-7.0.10/redis-cluster/7002
cp $HOME/software/redis-7.0.10/redis.conf $HOME/redis-7.0.10/redis-cluster/8002
---------------------------
# node C节点
mkdir -p $HOME/redis-7.0.10/redis-cluster/{7003,8003}
mkdir -p /data/redis-cluster/{7003,8003}
mkdir -p /data/redis-cluster/7003/{logs,data}
mkdir -p /data/redis-cluster/8003/{logs,data}
touch /data/redis-cluster/7003/logs/redis_7003.log
touch /data/redis-cluster/8003/logs/redis_8003.log
mkdir $HOME/redis-7.0.10/redis-cluster/conf
cp $HOME/software/redis-7.0.10/redis.conf $HOME/redis-7.0.10/redis-cluster/7003
cp $HOME/software/redis-7.0.10/redis.conf $HOME/redis-7.0.10/redis-cluster/8003
```



## 3.配置集群

### 3.1 node A节点

```shell
# 7001 redis.conf配置参数如下：
vi $HOME/redis-7.0.10/redis-cluster/7001/redis.conf
bind 0.0.0.0
protected-mode yes
port 7001
daemonize yes
pidfile /data/redis-cluster/redis_7001.pid
logfile "/data/redis-cluster/7001/logs/redis_7001.log"
dir /data/redis-cluster/7001/data
masterauth <New Password>
requirepass <New Password>
maxmemory 2g
appendonly yes
cluster-enabled yes
cluster-config-file nodes-7001.conf
cluster-node-timeout 30000
cluster-require-full-coverage no
---------------------------
# 8001 redis.conf配置参数如下：
vi $HOME/redis-7.0.10/redis-cluster/8001/redis.conf
bind 0.0.0.0
protected-mode yes
port 8001
daemonize yes
pidfile /data/redis-cluster/redis_8001.pid
logfile "/data/redis-cluster/8001/logs/redis_8001.log"
dir /data/redis-cluster/8001/data
masterauth <New Password>
requirepass <New Password>
maxmemory 2g
appendonly yes
cluster-enabled yes
cluster-config-file nodes-8001.conf
cluster-node-timeout 30000
cluster-require-full-coverage no
```



### 3.2 node B节点

```shell
# 7002 redis.conf配置参数如下：
vi $HOME/redis-7.0.10/redis-cluster/7002/redis.conf
bind 0.0.0.0
protected-mode yes
port 7002
daemonize yes
pidfile /data/redis-cluster/redis_7002.pid
logfile "/data/redis-cluster/7002/logs/redis_7002.log"
dir /data/redis-cluster/7002/data
masterauth <New Password>
requirepass <New Password>
maxmemory 2g
appendonly yes
cluster-enabled yes
cluster-config-file nodes-7002.conf
cluster-node-timeout 30000
cluster-require-full-coverage no
---------------------------
# 8002 redis.conf配置参数如下：
vi $HOME/redis-7.0.10/redis-cluster/8002/redis.conf
bind 0.0.0.0
protected-mode yes
port 8002
daemonize yes
pidfile /data/redis-cluster/redis_8002.pid
logfile "/data/redis-cluster/8002/logs/redis_8002.log"
dir /data/redis-cluster/8002/data
masterauth <New Password>
requirepass <New Password>
maxmemory 2g
appendonly yes
cluster-enabled yes
cluster-config-file nodes-8002.conf
cluster-node-timeout 30000
cluster-require-full-coverage no
```



### 3.3 node C节点

```shell
# 7003 redis.conf配置参数如下：
vi $HOME/redis-7.0.10/redis-cluster/7003/redis.conf
bind 0.0.0.0
protected-mode yes
port 7003
daemonize yes
pidfile /data/redis-cluster/redis_7003.pid
logfile "/data/redis-cluster/7003/logs/redis_7003.log"
dir /data/redis-cluster/7003/data
masterauth <New Password>
requirepass <New Password>
maxmemory 2g
appendonly yes
cluster-enabled yes
cluster-config-file nodes-7003.conf
cluster-node-timeout 30000
cluster-require-full-coverage no
---------------------------
# 8003 redis.conf配置参数如下：
vi $HOME/redis-7.0.10/redis-cluster/8003/redis.conf
bind 0.0.0.0
protected-mode yes
port 8003
daemonize yes
pidfile /data/redis-cluster/redis_8003.pid
logfile "/data/redis-cluster/8003/logs/redis_8003.log"
dir /data/redis-cluster/8003/data
masterauth <New Password>
requirepass <New Password>
maxmemory 2g
appendonly yes
cluster-enabled yes
cluster-config-file nodes-8003.conf
cluster-node-timeout 30000
cluster-require-full-coverage no
```



## 4.启动redis集群node并验证

### 4.1 启动所有节点

```shell
# node A节点
cd $HOME/redis-7.0.10/bin/
./redis-server $HOME/redis-7.0.10/redis-cluster/7001/redis.conf
./redis-server $HOME/redis-7.0.10/redis-cluster/8001/redis.conf
---------------------------
# node B节点
cd $HOME/redis-7.0.10/bin/
./redis-server $HOME/redis-7.0.10/redis-cluster/7002/redis.conf
./redis-server $HOME/redis-7.0.10/redis-cluster/8002/redis.conf
---------------------------
# node C节点
cd $HOME/redis-7.0.10/bin/
./redis-server $HOME/redis-7.0.10/redis-cluster/7003/redis.conf
./redis-server $HOME/redis-7.0.10/redis-cluster/8003/redis.conf
```

### 4.2 初始化redis集群

```shell
# 进入redis
cd $HOME/redis-7.0.10/bin
# 初始化redis集群配置及验证
./redis-cli -a "<New Password>" --cluster create node1:7001 node2:7002 node3:7003 node1:8001 node2:8002 node3:8003 --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica node2:8002 to node1:7001
Adding replica node3:8003 to node2:7002
Adding replica node1:8001 to node3:7003
M: ce5a2040427a2265e0b0fdc4b82ad837047b9ff6 node1:7001
   slots:[0-5460] (5461 slots) master
M: 54e084fd2a76be4a3d3b8f2609bd304b106ca79f node2:7002
   slots:[5461-10922] (5462 slots) master
M: af9c4d1374869acd50f932767044625b84e573bf node3:7003
   slots:[10923-16383] (5461 slots) master
S: 80aadc6de392cac358488d2396cfb02438d74219 node1:8001
   replicates af9c4d1374869acd50f932767044625b84e573bf
S: a840f873ba7ca15675755bcc5c723d8126c2380d node2:8002
   replicates ce5a2040427a2265e0b0fdc4b82ad837047b9ff6
S: 3a553fb61ac7df4a933017328d02f6eda8e94dca node3:8003
   replicates 54e084fd2a76be4a3d3b8f2609bd304b106ca79f
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node node1:7001)
M: ce5a2040427a2265e0b0fdc4b82ad837047b9ff6 node1:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 54e084fd2a76be4a3d3b8f2609bd304b106ca79f node2:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: af9c4d1374869acd50f932767044625b84e573bf node3:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 3a553fb61ac7df4a933017328d02f6eda8e94dca node3:8003
   slots: (0 slots) slave
   replicates 54e084fd2a76be4a3d3b8f2609bd304b106ca79f
S: a840f873ba7ca15675755bcc5c723d8126c2380d node2:8002
   slots: (0 slots) slave
   replicates ce5a2040427a2265e0b0fdc4b82ad837047b9ff6
S: 80aadc6de392cac358488d2396cfb02438d74219 node1:8001
   slots: (0 slots) slave
   replicates af9c4d1374869acd50f932767044625b84e573bf
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



## 5.配置启停脚本

### 5.1 node A节点脚本

```shell
# 启动脚本
vi $HOME/yunwei/redis_cluster_7001_8001_start.sh
#!/bin/bash
cd $HOME/redis-7.0.10/bin
./redis-server $HOME/redis-7.0.10/redis-cluster/7001/redis.conf
./redis-server $HOME/redis-7.0.10/redis-cluster/8001/redis.conf
---------------------------
# 停止脚本
vi $HOME/yunwei/redis_cluster_7001_8001_stop.sh
#!/bin/bash
cd $HOME/redis-7.0.10/bin
./redis-cli -p 7001 -a '<New Password>' shutdown
./redis-cli -p 8001 -a '<New Password>' shutdown
```



### 5.2 node B节点脚本

```shell
# 启动脚本
vi $HOME/yunwei/redis_cluster_7002_8002_start.sh
#!/bin/bash
cd $HOME/redis-7.0.10/bin
./redis-server $HOME/redis-7.0.10/redis-cluster/7002/redis.conf
./redis-server $HOME/redis-7.0.10/redis-cluster/8002/redis.conf
---------------------------
# 停止脚本
vi $HOME/yunwei/redis_cluster_7002_8002_stop.sh
#!/bin/bash
cd $HOME/redis-7.0.10/bin
./redis-cli -p 7002 -a '<New Password>' shutdown
./redis-cli -p 8002 -a '<New Password>' shutdown
```



### 5.3 node C节点脚本

```shell
# 启动脚本
vi $HOME/yunwei/redis_cluster_7003_8003_start.sh
#!/bin/bash
cd $HOME/redis-7.0.10/bin
./redis-server $HOME/redis-7.0.10/redis-cluster/7003/redis.conf
./redis-server $HOME/redis-7.0.10/redis-cluster/8003/redis.conf
---------------------------
# 停止脚本
vi $HOME/yunwei/redis_cluster_7003_8003_stop.sh
#!/bin/bash
cd $HOME/redis-7.0.10/bin
./redis-cli -p 7003 -a '<New Password>' shutdown
./redis-cli -p 8003 -a '<New Password>' shutdown
```



## 6.例行维护及迁移

### 6.1 例行维护操作

```shell
1.按顺序登录【node A节点、node B节点、node C节点】
2.按顺序依次执行停止脚本
3.等待变更及其他操作
4.按顺序依次执行启动脚本
```



### 6.2 测试检查redis状态

```shell
cd $HOME/redis-7.0.10/bin
./redis-cli --cluster check node1:7001 -a '<New Password>'
```



### 6.3 redis集群迁移操作

```shell
...(待续)
```

