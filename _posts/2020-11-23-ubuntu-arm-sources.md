---
layout: post
title: 树莓派4b Ubuntu 20.04 LTS 更换镜像源问题
category: Linux
tags: [Linux]
---

## 树莓派4b Ubuntu 20.04 LTS 更换镜像源问题

树莓派4B安装Ubuntu20.04，在安装成功后更改国内镜像源时使用了x86架构的镜像源  
如：  
阿里源：http://mirrors.aliyun.com/ubuntu  
清华源：https://mirrors.tuna.tsinghua.edu.cn/ubuntu/  

都导致了`sudo apt-get update` 时出现错误
```
E: Some index files failed to download, they have been ignored, or old ones used instead
```
## 解决方法
中科大镜像源的说明：：http://mirrors.ustc.edu.cn/help/ubuntu-ports.html

树莓派4B的架构要用的是ubuntu-ports的源，而不是ubuntu的源

修改/etc/apt/sources.list文件，推荐使用中科大和清华源

中科大源：
```
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal main main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
```

清华源：
```
deb https://mirror.tuna.tsinghua.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
deb https://mirror.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
deb https://mirror.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
deb https://mirror.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
deb https://mirror.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-proposed main restricted universe multiverse
```

