---
layout: post
title: Ubuntu server 18.04.1 LTS 修改主机名和网络配置
category: Linux
tags: [Linux]
---


## Ubuntu server 18.04.1 LTS 修改主机名

在Ubuntu server 16和更早版本中，只要修改/etc/hostname和/etc/hosts即可  
Ubuntu server 18中默认安装了Cloud-init，用于处理云实例的初始化   
sudo vim /etc/cloud/cloud.cfg

```shell
# This will cause the set+update hostname module to not operate (if true)
preserve_hostname: true
```

默认preserve_hostname: false，改完/etc/hostname和/etc/hosts重启reboot后，/etc/hostname又还原了！@后的主机名还是重启前的；  
而preserve_hostname: true时，修改/etc/hostname和/etc/hosts重启reboot后，新主机名生效了  

查看主机名
```shell
uname -a
```
主机名实际存储在/proc/sys/kernel/hostname,但是不能修改
```shell
cat /proc/sys/kernel/hostname
```


## Ubuntu server 18.04.1 LTS 网络配置

Ubuntu server 18采用netplan管理网络配置使用新的配置方法，修改/etc/network/interfaces不能配置网络

vim /etc/network/interfaces 修改网络配置时发现
```shell
# ifupdown has been replaced by netplan(5) on this system.  See
# /etc/netplan for current configuration.
# To re-enable ifupdown on this system, you can run:
#    sudo apt install ifupdown
```
那就直接配置netplan
```shell
cd /etc/netplan/
```
可能有的不是50-cloud-init.yaml，但只要是.yaml就可以的，netplan会扫描的

sudo vi /etc/netplan/50-cloud-init.yaml 
```shell
network:
    ethernets:
        ens33:
            addresses: [192.168.0.10/24]
            gateway4: 192.168.0.1
            nameservers:
                addresses: [114.114.114.114, 8.8.8.8]
            dhcp4: no
            optional: no
        ens160:
            addresses: []
            dhcp4: true
            optional: true
    version: 2
```
使用yaml语法格式，每个配置项使用空格缩进表示层级

sudo netplan apply
使用netplan apply生效配置，无需reboot

在看一下网络信息
ifconfig
```shell
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.10  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::20c:29ff:fe7b:b188  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:7b:b1:88  txqueuelen 1000  (以太网)
        RX packets 12882  bytes 1233165 (1.2 MB)
        RX errors 0  dropped 1  overruns 0  frame 0
        TX packets 72762  bytes 8857747 (8.8 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::20c:29ff:fe7b:b192  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:7b:b1:92  txqueuelen 1000  (以太网)
        RX packets 2  bytes 120 (120.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10  bytes 1573 (1.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (本地环回)
        RX packets 126088  bytes 14019433 (14.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 126088  bytes 14019433 (14.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
以上只是针对ubuntu18.04 Server版，对于18.04 desktop它缺省是使用NetworkManger来进行管理，可使用图形界面进行配置，其网络配置文件是保存在：/etc/NetworkManager/system-connections目录下的，跟Server版区别还是比较大的。
