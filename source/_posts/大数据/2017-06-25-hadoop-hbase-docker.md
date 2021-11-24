---
title: 基于Docker单机快速部署Hbase和Hadoop集群
date: 2017-06-25 00:00:00
author: robin zheng
img: https://hbase.apache.org/images/hbase_logo_with_orca_large.png
categories: 大数据
tags:
  - 大数据
  - Hbase
  - Hadoop
---

## 基于Docker单机快速部署Hbase和Hadoop集群

- 采用krejcmat/hadoop-hbase-docker项目
- Docker 需要先安装好

######相关产品版本

| system       | version    |
| ----------------|:-----------:|
| HBase        | 1.2.4     |
| Docker       | 17.03.0-ce  |

######repository 文件目录 
```
$ tree
.
├── hadoop-hbase-base
│   ├── Dockerfile
│   └── files
│       ├── bashrc
│       └── hbase-env.sh
├── hadoop-hbase-master
│   ├── Dockerfile
│   └── files
│       ├── hadoop
│       │   ├── configure-slaves.sh
│       │   ├── core-site.xml
│       │   ├── hdfs-site.xml
│       │   ├── mapred-site.xml
│       │   ├── run-wordcount.sh
│       │   ├── start-hadoop.sh
│       │   ├── start-ssh-serf.sh
│       │   ├── stop-hadoop.sh
│       │   └── yarn-site.xml
│       └── hbase
│           ├── hbase-site.xml
│           ├── start-hbase.sh
│           └── stop-hbase.sh
├── hadoop-hbase-slave
│   ├── Dockerfile
│   └── files
│       ├── hadoop
│       │   ├── core-site.xml
│       │   ├── hdfs-site.xml
│       │   ├── mapred-site.xml
│       │   ├── start-ssh-serf.sh
│       │   └── yarn-site.xml
│       └── hbase
│           └── hbase-site.xml
├── README.md
├── resize-cluster.sh
├── build-image.sh
└── start-container.sh

```

### 1] 克隆项目库到服务器
```bash
root@ubuntu:~# ls
configure-members.sh  hadoop-hbase-docker  start-container.sh
root@ubuntu:~# cd hadoop-hbase-docker
-bash: cd: cd: No such file or directory
root@ubuntu:~# git clone https://github.com/krejcmat/hadoop-hbase-docker.git
root@ubuntu:~# cd hadoop-hbase-docker
root@ubuntu:~/hadoop-hbase-docker# ls
build-image.sh  hadoop-hbase-base  hadoop-hbase-master  hadoop-hbase-slave  README.md  resize-cluster.sh  start-container.sh
```

### 2] 获得Docker镜像，从Docker Hub 直接Pull
```bash
root@ubuntu:~# docker pull krejcmat/hadoop-hbase-master:latest
root@ubuntu:~# docker pull krejcmat/hadoop-hbase-slave:latest
```
######Check images 检查镜像
```bash
root@ubuntu:~# docker images
REPOSITORY                           TAG                                                     IMAGE ID            CREATED             SIZE
krejcmat/hadoop-master               latest                                                  5089e2ba0492        16 months ago       671 MB
krejcmat/hadoop-slave                latest                                                  f41baab63271        16 months ago       671 MB
krejcmat/hadoop-hbase-slave          latest                                                  f7f8427fb25f        16 months ago       1.32 GB
krejcmat/hadoop-hbase-master         latest                                                  f2313bc4bd42        16 months ago       1.32 GB
```

### 3] 初始化Hadoop （主节点和从节点）

######a)run containers
start-container.sh 脚本第一个参数是镜像tag, 第二个是节点数量， 我们采用latest及三个节点

```bash
root@ubuntu:~/hadoop-hbase-docker#  ./start-container.sh latest 3

start master container...
start slave1 container...
start slave2 container...
```

#####Check status 检查集群情况
######查看Docker容器情况

```bash
root@ubuntu:~/hadoop-hbase-docker# docker ps -a
CONTAINER ID        IMAGE                                                                               COMMAND                  CREATED             STATUS                      PORTS                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            NAMES
cde44b48b3b8        andypetrella/spark-notebook:0.7.0-scala-2.10.6-spark-2.1.1-hadoop-2.7.2-with-hive   "bin/spark-notebook"     16 hours ago        Exited (129) 15 hours ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    jovial_kilby
7f0884c5f835        krejcmat/hadoop-hbase-slave:latest                                                  "/bin/sh -c ''/roo..."   41 hours ago        Up 41 hours                 0.0.0.0:32919->22/tcp, 0.0.0.0:32918->7373/tcp, 0.0.0.0:32917->7946/tcp, 0.0.0.0:32916->8030/tcp, 0.0.0.0:32915->8031/tcp, 0.0.0.0:32914->8032/tcp, 0.0.0.0:32913->8033/tcp, 0.0.0.0:32912->8040/tcp, 0.0.0.0:32911->8042/tcp, 0.0.0.0:32910->8060/tcp, 0.0.0.0:32909->8088/tcp, 0.0.0.0:32908->9000/tcp, 0.0.0.0:32907->50010/tcp, 0.0.0.0:32906->50020/tcp, 0.0.0.0:32905->50060/tcp, 0.0.0.0:32904->50070/tcp, 0.0.0.0:32903->50075/tcp, 0.0.0.0:32902->50090/tcp, 0.0.0.0:32901->50475/tcp   slave2.krejcmat.com
c6ce615a8a67        krejcmat/hadoop-hbase-slave:latest                                                  "/bin/sh -c ''/roo..."   41 hours ago        Up 41 hours                 0.0.0.0:32900->22/tcp, 0.0.0.0:32899->7373/tcp, 0.0.0.0:32898->7946/tcp, 0.0.0.0:32897->8030/tcp, 0.0.0.0:32896->8031/tcp, 0.0.0.0:32895->8032/tcp, 0.0.0.0:32894->8033/tcp, 0.0.0.0:32893->8040/tcp, 0.0.0.0:32892->8042/tcp, 0.0.0.0:32891->8060/tcp, 0.0.0.0:32890->8088/tcp, 0.0.0.0:32889->9000/tcp, 0.0.0.0:32888->50010/tcp, 0.0.0.0:32887->50020/tcp, 0.0.0.0:32886->50060/tcp, 0.0.0.0:32885->50070/tcp, 0.0.0.0:32884->50075/tcp, 0.0.0.0:32883->50090/tcp, 0.0.0.0:32882->50475/tcp   slave1.krejcmat.com
164b4915b678        krejcmat/hadoop-hbase-master:latest                                                 "/bin/sh -c ''/roo..."   41 hours ago        Up 41 hours                 0.0.0.0:32881->22/tcp, 0.0.0.0:32880->7373/tcp, 0.0.0.0:32879->7946/tcp, 0.0.0.0:32878->8030/tcp, 0.0.0.0:32877->8031/tcp, 0.0.0.0:32876->8032/tcp, 0.0.0.0:32875->8033/tcp, 0.0.0.0:32874->8040/tcp, 0.0.0.0:32873->8042/tcp, 0.0.0.0:32872->8060/tcp, 0.0.0.0:32871->8088/tcp, 0.0.0.0:32870->9000/tcp, 0.0.0.0:32869->50010/tcp, 0.0.0.0:32868->50020/tcp, 0.0.0.0:32867->50060/tcp, 0.0.0.0:32866->50070/tcp, 0.0.0.0:32865->50075/tcp, 0.0.0.0:32864->50090/tcp, 0.0.0.0:32863->50475/tcp   master.krejcmat.com
90c339403a66        monitoringartist/zabbix-db-mariadb:latest                                           "/run.sh"                2 months ago        Up 3 weeks                  0.0.0.0:3306->3306/tcp                                                                                                                                                                                                                                                                                                                                                                                                                                                                           zabbix-db
c724e2c7d6a5        tensorflow/tensorflow                                                               "/run_jupyter.sh"        3 months ago        Up 3 weeks                  6006/tcp, 0.0.0.0:8888->8888/tcp                                                                                                                                                                                                                                                                                                                                                                                                                                                                 amazing_einstein
```
######进入容器master.krejcmat.com Master节点

```bash
root@ubuntu:~/hadoop-hbase-docker# docker exec -it  master.krejcmat.com  bash
```
######Check members of cluster  查看集群成员
```bash
root@master:~# serf members
master.krejcmat.com  172.17.0.3:7946  alive
slave1.krejcmat.com  172.17.0.5:7946  alive
slave2.krejcmat.com  172.17.0.6:7946  alive
```

#####b)Run Hadoop cluster 启动Hadoop集群
######Creating configures file for Hadoop and Hbase(includes zookeeper)  
```bash
root@master:~# ./configure-slaves.sh

Warning: Permanently added 'slave2.krejcmat.com,172.17.0.6' (ECDSA) to the list of known hosts.
slaves                                                                                                                                                                   100%   60     0.1KB/s   00:00
Warning: Permanently added 'slave2.krejcmat.com,172.17.0.6' (ECDSA) to the list of known hosts.
slaves                                                                                                                                                                   100%   60     0.1KB/s   00:00
Warning: Permanently added 'slave2.krejcmat.com,172.17.0.6' (ECDSA) to the list of known hosts.
hbase-site.xml                                                                                                                                                           100% 1750     1.7KB/s   00:00
Warning: Permanently added 'slave1.krejcmat.com,172.17.0.5' (ECDSA) to the list of known hosts.
slaves                                                                                                                                                                   100%   60     0.1KB/s   00:00
Warning: Permanently added 'slave1.krejcmat.com,172.17.0.5' (ECDSA) to the list of known hosts.
slaves                                                                                                                                                                   100%   60     0.1KB/s   00:00
Warning: Permanently added 'slave1.krejcmat.com,172.17.0.5' (ECDSA) to the list of known hosts.
hbase-site.xml                                                                                                                                                           100% 1750     1.7KB/s   00:00
Warning: Permanently added 'master.krejcmat.com,172.17.0.3' (ECDSA) to the list of known hosts.
slaves                                                                                                                                                                   100%   60     0.1KB/s   00:00
Warning: Permanently added 'master.krejcmat.com,172.17.0.3' (ECDSA) to the list of known hosts.
slaves                                                                                                                                                                   100%   60     0.1KB/s   00:00
Warning: Permanently added 'master.krejcmat.com,172.17.0.3' (ECDSA) to the list of known hosts.
hbase-site.xml                                                                                                                                                           100% 1750     1.7KB/s   00:00
  
```

######Starting Hadoop 启动Hadoop
```
root@master:~# ./start-hadoop.sh 
 #For stop Hadoop ./stop-hadoop.sh
```

######Print Java processes
```bash
root@master:~# jps
4314 DataNode
4640 ResourceManager
5499 HRegionServer
16242 Main
26091 Jps
4477 SecondaryNameNode
5274 HQuorumPeer
5371 HMaster
6698 Main
4187 NameNode

```

######Print status of Hadoop cluster 
```
root@master:~# hdfs dfsadmin -report
Configured Capacity: 31304097792 (29.15 GB)
Present Capacity: 23839765476 (22.20 GB)
DFS Remaining: 23839056868 (22.20 GB)
DFS Used: 708608 (692 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (3):

Name: 172.17.0.5:50010 (slave1.krejcmat.com)
Hostname: slave1.krejcmat.com
Decommission Status : Normal
Configured Capacity: 10434699264 (9.72 GB)
DFS Used: 233472 (228 KB)
Non DFS Used: 2484862644 (2.31 GB)
DFS Remaining: 7949603148 (7.40 GB)
DFS Used%: 0.00%
DFS Remaining%: 76.18%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 9
Last contact: Sun Jun 25 14:52:10 UTC 2017


Name: 172.17.0.6:50010 (slave2.krejcmat.com)
Hostname: slave2.krejcmat.com
Decommission Status : Normal
Configured Capacity: 10434699264 (9.72 GB)
DFS Used: 233472 (228 KB)
Non DFS Used: 2484608692 (2.31 GB)
DFS Remaining: 7949857100 (7.40 GB)
DFS Used%: 0.00%
DFS Remaining%: 76.19%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 9
Last contact: Sun Jun 25 14:52:10 UTC 2017


Name: 172.17.0.3:50010 (master.krejcmat.com)
Hostname: master.krejcmat.com
Decommission Status : Normal
Configured Capacity: 10434699264 (9.72 GB)
DFS Used: 241664 (236 KB)
Non DFS Used: 2494860980 (2.32 GB)
DFS Remaining: 7939596620 (7.39 GB)
DFS Used%: 0.00%
DFS Remaining%: 76.09%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 9
Last contact: Sun Jun 25 14:52:10 UTC 2017

```

### 4] 初始化Hbase并执行Hbase shell
######Start HBase
```bash
root@master:~# ./start-hbase.sh

(hbase(main):001:0>)
```
######Check status
```bash
root@master:~# hbase shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hbase/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.1.3, r72bc50f5fafeb105b2139e42bbee3d61ca724989, Sat Jan 16 18:29:00 PST 2016

hbase(main):001:0> status
3 servers, 0 dead, 1.6667 average load
```
######Example of creating table and adding some values
```
hbase(main):014:1> create 'album','label','image'
```
Now you have a table called album, with a label, and an image family. These families are “static” like the columns in the RDBMS world.

Add some data:
```
hbase(main):016:1> put 'album','label1','label:size','10'
hbase(main):017:1> put 'album','label1','label:color','255:255:255'
hbase(main):018:1> put 'album','label1','label:text','Family album'
hbase(main):019:1> put 'album','label1','image:name','holiday'
hbase(main):020:1> put 'album','label1','image:source','/tmp/pic1.jpg'
```

Print table album,label1.
```
hbase(main):002:0* get 'album','label1'
COLUMN                                              CELL
 image:name                                         timestamp=1498252174789, value=holiday
 image:source                                       timestamp=1498252186627, value=/tmp/pic1.jpg
 label:color                                        timestamp=1498252152684, value=255:255:255
 label:size                                         timestamp=1498252138387, value=10
 label:text                                         timestamp=1498252164411, value=Family album
5 row(s) in 0.4150 seconds
```
