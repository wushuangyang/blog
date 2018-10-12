---
layout: post
title: RedHat6离线安装ansible过程
category: Linux运维
tags: [Linux, ansible]
---

## 离线安装ansible过程
在Linux虚拟机上，系统是RedHat6，这是一台无法连外网的离线的虚拟机，下载只能通过其他电脑连外网，然后将下载包上传到虚拟机，离线安装。

### 一、创建用户ansible
用root登录Linux虚拟机
```shell
useradd ansible
passwd ansible
```

### 二、安装ansible

以下用到的包都可以在[https://pkgs.org/download/](https://pkgs.org/download/) 搜索到，并可以下载，安装过程使用**root**权限

下载包：ansible-2.4.4.0-1.el6.ans.noarch.rpm，上传后安装
```shell
rpm -ivh ansible-2.4.4.0-1.el6.ans.noarch.rpm 
```
结果：
```
PyYAML is needed by ansible-2.4.4.0-1.el6.ans.noarch
python-jinja2 is needed by ansible-2.4.4.0-1.el6.ans.noarch
python-six is needed by ansible-2.4.4.0-1.el6.ans.noarch
sshpass is needed by ansible-2.4.4.0-1.el6.ans.noarch
```
PyYAML：
下载包：PyYAML-3.10-3.1.el6.x86_64.rpm，上传后安装
```shell
rpm -ivh PyYAML-3.10-3.1.el6.x86_64.rpm
``` 
结果：
```
libyaml-0.so.2()(64bit) is needed by PyYAML-3.10-3.1.el6.x86_64
```
在下载包：libyaml-0.1.3-4.el6_6.x86_64.rpm，上传后安装
```
rpm -ivh libyaml-0.1.3-4.el6_6.x86_64.rpm
```
没报错

PyYAML又回头安装，好像OK了


python-jinja2：

下载了：python-jinja2-2.2.1-3.el6.x86_64.rpm，上传后安装
```
rpm -ivh python-jinja2-2.2.1-3.el6.x86_64.rpm
```
结果：
```
python-babel >= 0.8 is needed by python-jinja2-2.2.1-3.el6.x86_64
```

下载了：python-babel-0.9.4-5.1.el6.noarch.rpm，上传后安装
```
rpm -ivh python-babel-0.9.4-5.1.el6.noarch.rpm
```
没报错

python-jinja2又回头安装，好像OK了

python-six：
下载了：python-six-1.9.0-2.el6.noarch.rpm，上传后安装
```
rpm -ivh python-six-1.9.0-2.el6.noarch.rpm
```
好像就OK了

sshpass：
下载了：sshpass-1.05-5.el6.art.x86_64.rpm，上传后安装
```
rpm -ivh sshpass-1.05-5.el6.art.x86_64.rpm
```
好像OK了

回头安装ansible：
```
rpm -ivh ansible-2.4.4.0-1.el6.ans.noarch.rpm
```
没报错！

验证一下：
```
ansible --version
```
结果：
```
ansible 2.4.4.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.6/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.6.6 (r266:84292, Oct 12 2012, 14:23:48) [GCC 4.4.6 20120305 (Red Hat 4.4.6-4)]
```
安装完毕！

### 三、小结
可能有的人安装过程中遇到的缺少依赖不一样，但解决方法相同。


配置文件ansible.cfg在/etc/ansible/中，该目录下有：ansible.cfg，hosts，roles目录
我把这三个文件都备份一下，我担心的没有root权限无法修改配置的问题，用root先改改，其实是多余的！
ansible读取配置文件的顺序：当前命令执行的路径->用户家目录下的.ansible.cfg->/etc/ansible.cfg，先找到哪个就使用哪个的配置，
其中ansible.cfg配置的所有内容均可在命令行通过参数的形式传递或者定义在playbooks中。所以可以简单认为，ansible安装好了就能直接用了！

