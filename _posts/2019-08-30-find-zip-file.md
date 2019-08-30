---
layout: post
title: 查找文件和归档文件
category: Linux运维
tags: [Linux, shell]
---

## 查找文件和归档文件

一个朋友找我说他的服务器上有很多日志文件，需要找出文件发送给他人，让我给他写给脚本。  
具体要求：文件在/wu/fileDir/下，文件名是随机数，没有规则，不要太久前的，也不要最近的文件，就要半个月前到三个月之内的文件。    
他又说他可以`ls -lrt`按时间排序，然后选出90天内，15天前的文件，但是太多了，一个一个下载太麻烦了。  

分析一下这里我们需要用到两个操作：  
- 查找文件
- 文件归档

### 查找文件
find
```bash
find path -option [ -print ] [ -exec -ok command ] {} \;
```

参数说明 :
find 根据下列规则判断 path 和 expression，在命令列上第一个 - ( ) , ! 之前的部份为`path`，之后的是`expression`。
如果`path`是空字串则使用目前路径，如果`expression`是空字串则使用`-print`为预设 expression。
expression 中最常用参数：  
-mount, -xdev : 只检查和指定目录在同一个文件系统下的文件，避免列出其它文件系统中的文件  
-amin n : 在过去n分钟内被读取过  
-cmin n : 在过去 n 分钟内被修改过  
-anewer file : 比文件 file 更晚被读取过的文件  
-atime n : 在过去n天内被读取过的文件(access time)  
-mtime n : 在过去n天内更改时间(modify time)  
-ctime n : 在过去n天内状态改动时间(change time)  
-cnewer file :比文件file更新的文件  
-empty : 空的文件  
-gid n or -group name : gid 是n或是 group 名称是 name  
-ipath p, -path p:路径名称符合p的文件，ipath 会忽略大小写  
-name name, -iname name : 文件名称符合name的文件。iname会忽略大小写  
-size n : 文件大小是n单位，b代表512位元组的区块，c表示字元数，k表示kilo bytes，w是二个位元组。  
-type c : 文件类型是c的文件。  
- d: 目录
- c: 字型装置文件
- b: 区块装置文件
- p: 具名贮列
- f: 一般文件
- l: 符号连结
- s: socket  

查找到符合要求的文件，我们只要执行
```bash
find /wu/fileDir/ -type f -mtime +15 -mtime -90
```


### 文件归档
可以使用zip或者tar等，朋友要zip包
```bash
zip [-dDqrS] [-b path] [zipfile [file ...]]
```
参数说明:
- -d : 从压缩文件内删除指定的文件
- -D : 压缩文件内不建立目录名称
- -q : 不显 示指令执行过程
- -r : 递归处理，将指定目录下的所有文件和子目录一并处理
- -S : 包含系统和隐藏文件
- -<压缩效率>压缩效率是一个介于1-9的 数值
- -b : 创建zip文件临时目录

归档文件file1、file2，打包wenjian.zip，可以简单执行
```bash
zip -r /wu/tmp/wenjian.zip file1 file2
```


那么最后使用一行命令完成:
```bash
find /wu/fileDir/ -type f -mtime +15 -mtime -90|xargs zip -r /wu/tmp/wenjian-`date +%Y%m%d`.zip
```
朋友在/wu/tmp/目录下开心地取走了wenjian-20190830.zip