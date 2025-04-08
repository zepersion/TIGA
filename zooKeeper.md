# zooKeeper

## 概述

zooKeeprer 是一个开源的分布式项目,为分布式相关应用提供协调服务的项目

从设计模式的角度来讲,是一个基于观察者模式的分布式服务管理框架,负责存储和管理大家都关心的数据,然后接受观察中的注册,一旦存储的数据状态发生改变,zookeeper就会负责通知已经在zookeeper上注册的那些观察者做出相关的反应

1.服务器端启动时去zookeeper注册信息

2.获取zookeeper中当前在线的服务器列表,并且注册监听

3.服务器节点下线

4.服务器节点上下线实践通知

## zookeeper特点

一个zookeeper只有一个领导者(leader),多个跟随者(follower)组成的集群

注意:集群中只要有半数以上(要求有2n+1个集群)的节点存活,Zookeeper集群就能正常服务

查询:全局数据一致性,每一个zookeeper服务都会保存一份相同的数据副本,客户端无论连接哪个zookeeper服务,数据都是一致的.

更新:

​     更新请求数据进行,来自同一个client(客户端)的更新请求按其发送顺序依次进行,

​     更新原子性:一次数据更新要么成功,要么失败

​     实时性:在一定时间范围内,clients能够读取最新的数据

## Zookeeper的数据结构

​     

Zookeeper数据类型的结构和Linux文件系统很类似,整体可以看作是一颗树,每个节点叫做一个ZNood,每个Znood默认能存储1mb的数据,

每个Znood都可以通过其路径作为唯一标识.

## Zookeeper部署

1.将安装包复制到容器内部

```shell
docker cp ./                   hadoop101:/opt/module
docker cp ./                   hadoop102:/opt/module
docker cp ./                   hadoop103:/opt/module
```

2.进入容器

```shell
docker exec -it hadoop101 /bin/bash
```

3.查看进入容器中的/opt/module中查看是否上传成功

```shell
cd /opt/module
ls
```

4.解压 

```shell
tar -xvf Zookeeper -C /opt/soft/
```

5.进入到配置目录

```shell
cd ./conf
```

6.重命名文件 

```shell
--将zoo_sample.cfg重命名为zoo.cfg
mv zoo_sample.cfg zoo.cfg

```

7.配置zoo.cfg

```shell
vi zoo.cfg
--#把datadir注释掉,在后面加
dataDir=/opt/soft/apeach    /zkData
:wq
```

8.创建zKdata目录

```shell
mkdir zkData
```

9.启动zookeeper

```
cd /opt/                       /bin          
./zkServer.sh start
```

10.查看状态

```shell
./zkServer.sh status
```

11.启动客户端

```shell
./zkCil.sh
```

12.退出客户端

```shell
quit
```

13.停止zookeeper服务

```shell
./zkServer stop
```

## 配置文件重要参数说明

1.ticKTime=2000 通信心跳数,zookeeper服务器和客户端心跳的时间,单位:毫秒(ms)(确定服务端是否运行alive)

心跳机制:检查是否存活

initlimit=10 初始通信时间限制.

集群中Follower(跟随者)服务器和leader(引导者)服务器之间初始连接时限能够容忍的最多心跳数.(发送十次未回复说明判断已经下线)

synclimit= 5 LF同步通信时限(5个心跳的时间)

集群中Leader和Follower之间中最大响应时间单位.

dataDir

数据文件的目录和数据持久化的路径

cilentport=2181 zookeeper客户端链接端口,客户端监听的端口.

## zookeeper

在单机安装的基础上继续配置

1.在data目录创建myid的文件

```
 cd /opt/soft/apache-zookeeper-3.8.4-bin/zkData/
touch myid
```

2.在myid添加内容 各自的编号

hadoop101 编号 101

hadoop102 编号 102

hadoop103 编号 103

```
hadoop101 
```

3.修改路径

```
opt/soft/apache-zookeeper-3.8.4-bin/conf
--修改绝对路径
```

4.添加内容

```
server.101=hadoop101:2888:3888

server.102=hadoop101:2888:3888

server.103=hadoop103:2888:3888



server .101=192.168.10.101:2181:2182
```



```shell
--查地址 docker inspect hadoop101
```

server .101=192.168.10.101:2181:2182



```
scp -r apach-.... /     hadopp102:/opt/soft
```

5.依次修改Hadoop102和Hadoop103服务器中的内容

6.

```
export ZOOKEEPER_HOME=/opt/soft/apache-zookeeper-3.8.4-bin
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

7.依次复制

```shell
scp -r /etc/profile hadoop102:/etc/profile
```

## 常用命令

1.进入客户端

```shell
zkCil.sh
```

2.显示所有可操作的命令

```
help
```

3.查看当前znode中所包含的内容

```shell
ls
```

4.创建两个普通节点

```shell
create /vcit 'xiaobai'
creat  /vcit 'xiaohei'

```

5.查看当前路径

```
listquota \
```

6.删除单个节点

```
delete /school /student01
```

7修改节点

```shell
Set /school 'qinghua'
```

8.删除所有节点

```
deleteall /school
```

9.查看节点状态

```
stat /school
```

