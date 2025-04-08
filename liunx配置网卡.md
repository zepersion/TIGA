# Liunx配置网卡



## 1.关闭防火墙

```shell
systemctl stop firewalld
```

## 2.使防火墙不可使用

```shell
systemctl enable firewalld
```

关闭防火墙

## 3.解压jdk

安装jdk
配置静态ip

1. 网卡配置的文件

  ```shell
   vi /etc/sysconfig/network-scripts/ifcfg-ens33
  ```

  lo回环地址：127.0.0.1
  原有内容配置

复制 

```shell
cp ifcfg-ens33 ifcfg-ens33.bak
```

注意：在部署任何东西前先备份一份

需要改动的内容有

```shell
//dhcp动态Ip
//static 静态
BOOTPROTO=“static'
iPADDR='----自己的地址‘
--网关
GATEWAY=""192.168.105.(0-255)"
--子网掩码
NETMASK“255.255.255.0”

---静态ip地址
--是否激活网卡
ONBOOT='yes'
```

更改后的内容是

```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
iPADDR="192.168.229.128"
GATEWAY="192.168.229.128"
NETMASK="255.255.255.0"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="b9225bbf-b39b-45d4-99a1-90cdce03243a"
DEVICE="ens33"
ONBOOT="yes"
~                                                                                                                                                                                            
~                  
```

2.重启网卡

```shell
systemctl restart network
```

