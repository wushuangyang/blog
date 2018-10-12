---
layout: post
title: VMware ESXi6.5.0安装
category: Linux运维
---


## VMware ESXi 6.5.0 安装

准备学习使用ESXi，弄到了一台二手组装机，我关注到的配置
> 华硕主板P8z77-vle
> 网卡芯片板载RealtekRTL8111F
> CPU Intel(R) Core(TM) i7-3770 3.40GHz
> 内存8G DDR3，品牌没看
> 硬盘1T，品牌没看

除了内存小了一点，当服务器还不错的

### 开始准备安装系统：
下载：VMware-VMvisor-Installer-6.5.0.update01-5969303.x86_64.iso

下载系统盘制作工具：http://rufus.akeo.ie/
使用 rufus-2.14.exe 工具制作U盘启动盘  
![rufus]({{site.baseurl}}/images/rufus.jpg)

U盘准备好后，插在二手机上，按电脑电源按钮后，马上按DEL，修改启动顺序，简单方法就是把U盘拖到第一个，F10保存并重启  
如下图所示
![bios]({{site.baseurl}}/images/bios1.jpg)

好了，开始安装VMware ESXi 6.5.0
![install1]({{site.baseurl}}/images/install1.jpg)

然后下一步，不详细写每一步了

然而不幸的事，无法识别网卡
![no-network]({{site.baseurl}}/images/no-network-adapters.jpg)

VMware ESXi 6.5.0 中没有RealtekRTL8111F驱动，原谅我没有品牌服务器，那些一般官网可以直接下载定制版的ESXI，然而这个二手组装机，真是伤心！

第一次，我放弃了，因为当时没有外网，就换个U盘重装了一个Ubuntu server 14，随便玩玩了。后来觉得这样简直是暴殄天物，就重新拾起ESXi！

### 重新拾起ESXi

参考网上教程，还是能找到RealtekRTL8111F网卡驱动的
网卡驱动从这里下载：https://vibsdepot.v-front.de/wiki/index.php/List_of_currently_available_ESXi_packages

找到对应的型号。点击进入下一页下载。

需要使用如下工具：

ESXi-Customizer-v2.7.2

安装好ESXi-Customizer-v2.7.2，将下载的VMware-VMvisor-Installer-6.5.0.iso和网卡驱动加载到软件中，`run`！
就可以得到一个新的包含新的网卡驱动的VMware-VMvisor-Installer-6.5.0.iso

![Customizer]({{site.baseurl}}/images/ESXi-6.0-Customizer.png)

再次重头再来，重新制作启动U盘，顺利安装了!

### 配置
主要是配置网络，F2进入配置界面；F12关闭/重启系统，F2关闭确认键，F11重启确认键；Enter保存键，ESC取消键或退出键。
![setting2]({{site.baseurl}}/images/setting2.png)


VMware ESXi 6.5.0的web页面不错，不用再安装VMware vSphere Client，我也没安装
直接浏览器登录就ok
![web1]({{site.baseurl}}/images/web1.png)
使用用户名（root）和安装时设置的密码进行登录
可以看到有个过期提示
![guoqi1]({{site.baseurl}}/images/guoqi1.png)
```
当前正在评估模式下使用ESXi。此许可证将在60天后过期。
```
依次点击 管理 -> 许可 -> 分配许可证，输入可用的许可证号码即可
**许可证**贵不贵？怎么弄到的？呵呵呵就找来了，就不分享了。

## VMware ESXi 6.5.0 使用

安装完就开始用用呗,快捷的方法当然是把虚拟机vmdk直接拷贝进去

我有一个现成虚拟机。在vmware workstation平台上能运行。
但我导入到VMware VSphere ESXI6.5.0上运行，开启电源失败，提示如下信息：

```  
模块 DevicePowerOn 打开电源失败。  
无法为 scsi0:0“/vmfs/volumes/XXXXX.vmdk” 创建虚拟 SCSI 设备  
无法打开磁盘 scsi0:0: 磁盘类型 2 不受支持或无效。请确保磁盘已导入。
```
这是因为磁盘的虚拟格式不一致，需要转换

查询官方资料后发现：
在VMware Workstation，VMware Fusion 或VMware Player平台上运行的虚拟机如果需要在Vmware ESX主机上运行，必须用Vmware vCenter Converter工具转换成ESX主机兼容的格式。
当然，如果虚拟机的磁盘镜像文件如果已经被导入到ESXi主机，则可以使用vmkfstools 工具手动将磁盘格式进行转换。

命令如下：
打开ESXi的SSH通道，并通过SSH登录至ESXi主机
```shell
vmkfstools -i <HostedVirtualDisk>  <ESXVirtualDisk>
```
所以，我需要使用vmkfstools工具将XXXXX.vmdk文件转换成ESXi主机兼容的格式：

```
cd /vmfs/volumes/51dc3538-bbdf69dc-6e61-782bcb765b0f/XXXXX/
vmkfstools -i XXXXX.vmdk  XXXXX-new.vmdk -d thin
```
备注：XXXXX-new.vmdk就是转换后的磁盘名字
-d选项为：使用精简置备模式。节省空间。
旧文件可以删除。



