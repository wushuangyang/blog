---
layout: post
title: 大量文件分开打包
category: Linux运维
tags: [Linux, shell]
---

朋友需要将某服务器上大量日志打包下载，但日志文件个数非常大，日志名称没有规律，总体积超过60G，虽然该服务器空间超过500G，但是备份服务器空间小，不方便转存，求助能否写个脚本，将这些日志文件分多个包，最好压缩一下，方便他下载。

**要求不多，安排!**


### 先写目录和包名前缀
```shell
#文件源目录
source_dir=/home/test/logs;
#打包生成目录
pack_dir=/home/test/tmp;
#包名前缀
pack_name="file-"
```

### 包分几个？想要几个自己输入
```shell
#分包数量
if [ $# -eq 0 ]; then
	echo "Please input the package num:"
	read -p "package num:" pack_num
else
	pack_num=$1;
fi
```
### 找到文件
```shell
cd $source_dir;
#所有文件名
file_dir=`ls -1 ${source_dir}`
```
### 怎么分呢？要算算
假如有300个文件，打3个包，每个包100个文件，压缩包file-0.tar.gz,file-1.tar.gz,file-2.tar.gz，完美！  
要是301个文件呢？可不能拆文件，那就file-0.tar.gz中101个文件，file-1.tar.gz中100个，file-2.tar.gz中100个，也完美。

```shell
#文件源目录中文件数量
total_num=`ls ${source_dir}|wc -w`;

#判断是否整除，不能整除的靠前放，计算每个包文件数量
if [ `expr ${total_num} % ${pack_num}`  =  "0" ] ;then
	each_pack_num=`echo "sclae=0; ${total_num}/${pack_num}"|bc`;
else
	each_pack_num=`echo "sclae=0; ${total_num}/${pack_num} + 1"|bc`;
fi
```
### 打包
分好个数了，就是打包了，我想起了九九乘法表，最好理解的就是双层for循环，在所有文件名中，循环取出文件名
```shell
#分组打包
for ((i=0 ; i < ${pack_num} ; i++)) ; do
    #文件名位置游标
	indx=$(expr $i \* $each_pack_num + 1);
    #取出的文件名列表
	file_list="";
	for ((j = 0 ; j < $each_pack_num ; j++));do
		file_name=`echo ${file_dir}|awk -v indx2=$indx '{print $indx2}'`
		file_list="$file_list $file_name"
		((indx=$indx + 1));
	done;

	tar -zcvf $pack_dir/$pack_name${i}.tar.gz $file_list;
done;
```

### 完整脚本
```shell
#!/bin/bash

#文件源目录
source_dir=/ibapp/patch/CCS;
#打包生成目录
pack_dir=/ibapp/patch;
#包名前缀
pack_name="file-"

#分包数量
if [ $# -eq 0 ]; then
	echo "Please input the package num:"
	read -p "package num:" pack_num
else
	pack_num=$1;
fi

#文件源目录中文件数量
total_num=`ls ${source_dir}|wc -w`;

#判断是否整除，不能整除的靠前放，计算每个包文件数量
if [ `expr ${total_num} % ${pack_num}`  =  "0" ] ;then
	each_pack_num=`echo "sclae=0; ${total_num}/${pack_num}"|bc`;
else
	each_pack_num=`echo "sclae=0; ${total_num}/${pack_num} + 1"|bc`;
fi

#所有文件名
file_dir=`ls -1 ${source_dir}`

cd $source_dir;

#分组打包
for ((i=0 ; i < ${pack_num} ; i++)) ; do
	echo "start package $pack_name${i}.tar.gz ..."
    #文件名位置游标
	indx=$(expr $i \* $each_pack_num + 1);
    #取出的文件名列表
	file_list="";
	for ((j = 0 ; j < $each_pack_num ; j++));do
		file_name=`echo ${file_dir}|awk -v indx2=$indx '{print $indx2}'`
		file_list="$file_list $file_name"
		((indx=$indx + 1));
	done;

	tar -zcvf $pack_dir/$pack_name${i}.tar.gz $file_list;
	if [ $? -eq 0 ] ; then
		echo "package $pack_name${i}.tar.gz completely!"
	else
		echo "package $pack_name${i}.tar.gz failed!"
		exit;
	fi

done;

echo "package completely!"
```