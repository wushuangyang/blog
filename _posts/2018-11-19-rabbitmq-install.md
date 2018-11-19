---
layout: post
title: Rabbitmq离线安装步骤
category: Linux运维
tags: [Linux, ansible]
---

## rabbitmq安装步骤

### 环境条件
Linux虚拟机上，系统RedHat6，无法连外网

上传文件
> rabbitmq-server-3.7.3-1.el6.noarch.rpm
> erlang-20.3.8.9-1.el6.x86_64.rpm
> compat-readline5-5.2-17.1.el6.x86_64.rpm
> socat-1.7.2.1-2.el6.art.x86_64.rpm
> openssl-1.0.1e-57.el6.x86_64.rpm

用root用户执行
```shell
rpm -ivh openssl-1.0.1e-57.el6.x86_64.rpm --force
rpm -ivh erlang-20.3.8.9-1.el6.x86_64.rpm

rpm -ivh compat-readline5-5.2-17.1.el6.x86_64.rpm
rpm -ivh socat-1.7.2.1-2.el6.art.x86_64.rpm

rpm -ivh rabbitmq-server-3.7.3-1.el6.noarch.rpm
```

unix文件打开数限制调整
```shell
vi /etc/security/limits.conf
```
文件末尾追加以下内容：
```
* soft nofile 65536
* hard nofile 65536

```

修改启动配置文件
```shell
vi /etc/init.d/rabbitmq-server
```
寻找行：PID_FILE=/var/run/rabbitmq/pid
在以上内容之下，插入行
export RABBITMQ_MNESIA_BASE=/rabbitmq/mnesia


补缺失的目录
```shell
cd /var/run/
mkdir rabbitmq
chown rabbitmq:rabbitmq rabbitmq

mkdir /rabbitmq
chown rabbitmq:rabbitmq /rabbitmq
```

启动RabbitMQ
使用root用户登录应用服务器，并执行以下命令
```shell
service rabbitmq-server start
```

激活WEB插件
```shell
rabbitmq-plugins enable rabbitmq_management
```

使用root用户登录应用服务器，并执行以下命令
```shell
chmod 644 /etc/rabbitmq/enabled_plugins
```

使用rabbitmq用户登录应用服务器，并执行以下命令
```shell
su rabbitmq
rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator
```

用admin进入rabbitmq后管页面，http://IP:15672

点击admin
点击Virtual Hosts
Add a new virtual hosts name框输入 /0001 点击 Add virtual host
点击/0001 修改user为admin