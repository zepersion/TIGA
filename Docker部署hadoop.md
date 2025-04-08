# Docker部署hadoop

## 1.安装好docker

## 2.网络规划

例如使用 192.168.10.0作为子网,192.168.10.254作为网关

```shell
docker network create --subnet 192.168.10.0/24 --gateway 192.168.10.254 big-date
```



| 节点      | ip             | 说明     |
| --------- | -------------- | -------- |
| hadoop100 | 192.168.10.100 | 基础镜像 |
| hadoop102 | 192.168.10.101 | 节点1    |
| hadoop103 | 192.168.10.102 | 节点2    |
| hadoop104 | 192.168.10.103 | 节点3    |

## **3**构建基础镜像

加载镜像

```shell
docker pull centos:centos7
```

构建镜像的命令

```shell
 docker run -idt --name hadoop100  --net big-date --ip 192.168.10.100 --hostname hostname --privileged centos:centos7 /usr/sbin/init
```

说明:**有两个权限操作分别是:     ** **--privileged** *和*/usr/sbin/init**

进入容器

```shell
docker exec -it hadoop100 /bin/bash
```

yum源设置

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

更新软件源

```shell
yum install -y epel-release
yum update -y
```

安装软件

1.安装和配置ssh

```shell
--安装ssh服务端
yum install -y openssh-server
--启动ssh
systemctl start sshd
--设置开机自启
systemctl enable sshd
```

2配置ssh



```shell
vi/etc/ssh/ ssh_config

```

UseDNS NO(去掉#,yes变成No)

rootlogin 去掉#

emptypasswords no(不允许空密码登录)

passwordAuth(认证)yes

3.重启ssh

```shell
systemctl restart sshd
```

### 安装网络工具

```shell
yum install net-tools
```

### 配置host文件

```shell
vi /etc/hosts
192.168.10.100 hadoop100
192.168.10.101 hadoop101
192.168.10.102 hadoop102
192.168.10.103 hadoop103


```

将容器打包为镜像

```shell
docker commit --message "基础镜像:添加ssh,net-tools工具" hadoop100 vcit/hadoop100:v0.1
```

查看镜像

```shell
docker images
```

##    4.创建容器

```shell
docker run -id --name hadoop101 --net big-date --ip 192.168.10.101 --hostname hadoop101 --privileged vcit/hadoop100:v0.1 /usr/sbin/init
docker run -id --name hadoop102 --net big-date --ip 192.168.10.102 --hostname hadoop101 --privileged vcit/hadoop100:v0.1 /usr/sbin/init
docker run -id --name hadoop103 --net big-date --ip 192.168.10.103 --hostname hadoop101 --privileged vcit/hadoop100:v0.1 /usr/sbin/init
```

![image-20250225102932065](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20250225102932065.png)

#### 进入101容器

```shell
docker exec -it hadoop101 /bin/bash
```

#### 是否可以Ping通其他容器

```shell
ping 192.168.10.102
ping 192.168.10.103
```

#### 设置当前密码

```shell
passwd
```

![image-20250225103223419](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20250225103223419.png)

#### 配置免密登录

##### a.生成密钥ssh key

```shell
ssh-keygen -t rsa
```

两次回车

![image-20250225103531201](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20250225103531201.png)

##### 查看公钥

```shell
cat ~/.ssh/id_rsa.pub
```

![image-20250225104306767](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20250225104306767.png)

##### 下载ssh客户端

```shell
yum install -y openssh-clients
```

##### 复制到其他容器

```shell
##安装openssh-clients

##复制容器的root密码 123456
 ssh-copy-id -i ~/.ssh/id_rsa.pub root@hadoop102
 ssh-copy-id -i ~/.ssh/id_rsa.pub root@hadoop103
```

**ps:以上操作需要每一个容器进行相同的操作**

```shell
ssh 主机名
```

## 5.安装jdk

```shell
mkdir -p /opt/module
mkdir -p /opt/software
```

软件压缩包或者安装包存储到module中

软件安装 到software中

##### 将宿主主机在内容复制到容器hadoop101 中

```shell
docker cp /opt/soft/jdk-8u212-linux-x64.tar.gz   hadoop101:/opt/module/

```

##### 进入容器看是否解压成功

```

```

解压到/opt/software/中

```shell
--解压
tar -xvf jdk                   -c /opt/software
--查看目录
```

##### 配置环境变量

```shell
--编辑环境变量的文件
vi /etc/profile
--文件后添加内容
export JAVA_HOME=/opt/software/jdk1.8.2_212
export PATH=$JAVA_HOME/bin:$PATH
--保存退出后重新加载环境变量文件
source etc/profile
--查看jdk是否配置成功
java -version
```

```shell
--找根目录中的**~/.bashrc**
--在最后一行加入 source etc/profile
```

##### 将hadoop101中的内容复制到hadoop102和Hadoop103容器中

```shell
scp -r /opt/module/ hadoop102:/opt/module
scp -r /etc/profile  hadoop102:/etc
scp -r ~/.bashrc  hadoop102:/root/
```

##### 重启容器,测试java是否正常

```shell
docker restart hadoop101
docker restart hadoop102
docker restart hadoop103

```

进入其中任意一个

```shell
java -version
```

## Hadoop安装部署

#### a解压



```shell
tar -xvf hadoop-3.1.3.tar.gz  -C /opt/soft/
```

解压的路径就是安装软件的路径

#### b配置环境变量

```shell
vi /etc/profile
```

文件配置的内容:

```shell
export HADOOP_HOME=/opt/soft/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin:HADOOP_HOME/sbin
--检查时候用cat
--保存退出
--重新加载配置环境变量文件
source /etc/profile

```

#### c 验证环境变量是否成功

```shell
hadoop version
```

启动docker(开机自启)

```shell
systemctl enable docker--开机自启
systemctl disenable docker --取消开机自启
```

![image-20250226142914008](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20250226142914008.png)

#### d 配置hadoop的配置文件

分别是:

##### core-site.xml

```shell
<configuration>
<!--对于namenode通过web端访问的地址,hadoop101作为主节点-->
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hadoop101:8020</value>
        </property>
       --namenode的地址
       <property>
                <name>hadoop.tmp.dir</name>
                <value>/opt/soft/hadoop-3.1.3/data</value>
        </property>
</configuration>
~                 
```

##### hdfs-site.xml

```shell
<configuration>
        <property>      
        <namenode>dfs.namenode.http-address</name>
        <value>hadoop101:9870</value>--namenode访问地址web
        </property>
                <property>
                <name>dfs.namenode.secondary.http-address</name>
               <value>hadoop102:9868</value>
</property>--secondary的web访问地址

</configuration>

```

##### yarn-site.xml

```shell
<configuration>

<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.nodemanager.aux--services</name>--指定mapreduce走shuffle模式
                <value> mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>hadoop101</value>
        </property>
        --管理资源的主机
        <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <value>JAVA_HOME,HADOOP_HOME</value>--环境变量
        </property>
Type  :quit<Enter>  to exit Vim

```

```
<configuration>
        <property>
                <name>mapreduce.freamework,name</name>
                <value> yarn</value>
        </property>
<tconfiguration>

```

##### 配置wokers

```shell
localhost
hadoop101
hadoop102
hadoop103

```

##### hadoop_env

```
export JAVA_HOME=/opt/soft/jdk-3.1.3
export HDFS_DATANODE_USER=root
export HADOOP_SECURE_DN_USER=hdfs
export HDFS_NAMENODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root

```

复制

```

```

