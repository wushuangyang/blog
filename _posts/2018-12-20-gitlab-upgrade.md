---
layout: post
title: Gitlab版本升级
category: Gitlab
tags: [Gitlab]
---


## gitlab版本升级
Ubuntu 14.04想升级gitlab，由11.2.1-ce到11.5.2-ce

### 开始入坑

找到清华源 https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/bionic/main/g/gitlab-ce/gitlab-ce_11.5.2-ce.0_amd64.deb
下载上传：gitlab-ce_11.5.2-ce.0_amd64.deb
执行：
```bash
sudo dpkg -i gitlab-ce_11.5.2-ce.0_amd64.deb
```

```
正在将 gitlab-ce (11.5.2-ce.0) 解包到 (11.2.1-ce.0) 上 ...
正在设置 gitlab-ce (11.5.2-ce.0) ...
/opt/gitlab/embedded/bin/ruby: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.25' not found (required by /opt/gitlab/embedded/lib/libruby.so.2.4)
dpkg: 处理软件包 gitlab-ce (--install)时出错：
 子进程 已安装 post-installation 脚本 返回了错误号 1
在处理时有错误发生：
 gitlab-ce
```
然后就退出了，gitlab也不能用了！

### 爬出坑

查资料才知道，下错版本了！
小伙伴们看好了

|序号|代号|系统版本|
|-|-|-|
|1|ol/7|Centos 7.x|
|2|ol/6|Centos 6.x|
|3|ubuntu/bionic|Ubuntu 18.04|
|4|ubuntu/xenial|Ubuntu 16.04|
|5|ubuntu/trusty|Ubuntu 14.04|

前面下载了bionic版的，这在Ubuntu 14.04用不了，应该下载trusty版，
另外我还没有测试过Ubuntu 18.04是不是可以用trusty，以后再试试吧。

重新下载：
https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/trusty/main/g/gitlab-ce/
这里的gitlab-ce_11.5.2-ce.0_amd64.deb

还是有
> /opt/gitlab/embedded/bin/ruby: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.25' not found (required by /opt/gitlab/embedded/lib/libruby.so.2.4)
dpkg: 处理软件包 gitlab-ce (--install)时出错：
 子进程 已安装 post-installation 脚本 返回了错误号 1


我去备机上看看/opt/gitlab/embedded/lib/libruby.so.2.4到底是啥？
```bash
ls -l /opt/gitlab/embedded/lib/libruby.so.2.4
```
```bash
lrwxrwxrwx 1 root root 16  8月 23 07:09 /opt/gitlab/embedded/lib/libruby.so.2.4 -> libruby.so.2.4.4
```
原来是一个软链接

在看看软链接的源：
```bash
#/opt/gitlab/embedded/lib
lrwxrwxrwx  1 root root       16  8月 23 07:09 libruby.so -> libruby.so.2.4.4
lrwxrwxrwx  1 root root       16  8月 23 07:09 libruby.so.2.4 -> libruby.so.2.4.4
-rwxr-xr-x  1 root root 15042863  8月 23 07:09 libruby.so.2.4.4
```
错误安装将libruby.so.2.4.4升级到了libruby.so.2.4.5

没想到好办法，就将libruby.so.2.4.4从备机上拷贝过来了,修改软链接。
```bash
cd /opt/gitlab/embedded/lib
sudo rm -rf libruby.so libruby.so.2.4
sudo ln -s libruby.so.2.4.4 libruby.so
sudo ln -s libruby.so.2.4.4 libruby.so.2.4.4
```
gitlab重新加载配置
```bash
sudo gitlab-ctl reconfigure
```
还好，还能用!

### 转身起飞

在 https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/trusty/main/g/gitlab-ce/
还看到还有新的gitlab-ce_11.5.4-ce.0_amd64.deb，就下载了新的

先停：
```bash
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq
```
安装升级包：
```bash
sudo dpkg -i gitlab-ce_11.5.4-ce.0_amd64.deb
```
启动：
```bash
sudo gitlab-ctl start
```
重新加载配置：
```bash
sudo gitlab-ctl reconfigure
```
OK了！
