---
layout: post
title: Tomcat使用https协议
category: Linux
tags: [Linux]
---

## Tomcat使用https协议

相较于http，https提供了身份验证与加密通信方法。出于安全性考虑，越来越多的网站开始使用https协议。  
要想使用https,首先,我们需要有SSL证书,证书可以通过两个渠道获得:  
#### 1.公开可信认证机构  
例如CA,但是申请一般是收费的，我们就是测试使用就先不考虑了。  
#### 2.自己生成证书
目前证书有以下常用文件格式：  
JKS(.keystore)，微软(.pfx)，PEM(.key + .crt)。其中,tomcat使用JKS格式,nginx使用PEM格式。  

JKS格式证书生成
需要jdk，可以在本地生成，也可以在服务器上生成，如果是本地生成的，需要将证书上传到服务器
```shell
keytool -genkey -v -alias tomcatKey -keyalg RSA -validity 3650 -keystore /home/wu/keyfile/tomcat.keystore
```
参数说明：  
alias: 别名，tomcatKey  
keyalg: 证书算法，RSA  
validity：证书有效时间，10年  
keystore：证书生成的目标路径和文件名，/wu/keyfile/tomcat.keystore  

还需要输入一些信息,重要的是秘钥库口令和秘要口令,测试时写了abcd1234

命令确认回车后将显示并需要输入
```shell
输入密钥库口令:  
再次输入新口令: 
您的名字与姓氏是什么?
  [Unknown]:  192.168.1.100
您的组织单位名称是什么?
  [Unknown]:  tomcat
您的组织名称是什么?
  [Unknown]:  tomcat
您所在的城市或区域名称是什么?
  [Unknown]:  shanghai
您所在的省/市/自治区名称是什么?
  [Unknown]:  shanghai
该单位的双字母国家/地区代码是什么?
  [Unknown]:  cn
CN=192.168.1.100, OU=tomcat, O=tomcat, L=shanghai, ST=shanghai, C=cn是否正确?
  [否]:  y

正在为以下对象生成 2,048 位RSA密钥对和自签名证书 (SHA256withRSA) (有效期为 3,650 天):
         CN=192.168.1.100, OU=tomcat, O=tomcat, L=shanghai, ST=shanghai, C=cn
输入 <ibappKey> 的密钥口令
        (如果和密钥库口令相同, 按回车):  
再次输入新口令: 

```
注意点“名字与姓氏”是服务器IP或者是域名，随意填写的话，后来浏览器访问会报警“服务器的证书和网址不符”！
准备好后：
```
[正在存储/home/wu/keyfile/tomcat.keystore]
输入密钥库口令:  
存储在文件 /home/wu/keyfile/tomcat.keystore> 中的证书
```

tomcat/conf/server.xml的配置，测试环境端口8088，redirectPort="8443"，但是8443端口配置被注释掉了
```xml
    <Connector URIEncoding="UTF-8" port="8088" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

所以只要在tomcat/conf/server.xml中添加https的配置，端口使用了8443，使用生成的证书ibapp.keystore
```xml
    <Connector URIEncoding="UTF-8" port="8443" protocol="org.apache.coyote.http11.Http11Protocol" 
               connectionTimeout="20000" maxThreads="300" acceptCount="100"
               SSLEnabled="true" scheme="https" secure="true" clientAuth="false" sslProtocol="TLS"
               keystoreFile="/home/wu/keyfile/tomcat.keystore" keystorePass="abcd1234"/>

```

在tomcat/conf/web.xml中添加，只运行https，不允许http

```xml
    <security-constraint>  
        <!-- Authorization setting for SSL -->  
        <web-resource-collection>  
            <web-resource-name>sslwebsokect</web-resource-name>  
            <url-pattern>/*</url-pattern>  
        </web-resource-collection>  
        <user-data-constraint>  
            <transport-guarantee>CONFIDENTIAL</transport-guarantee>  
        </user-data-constraint>  
    </security-constraint> 
```

重启tomcat

浏览器地址栏输入：http://192.168.1.100:8088/test
将会跳转到：https://192.168.1.100:8443/test

测试无异常，ok

#### 让客户端信任服务器证书

把服务器证书导出为一个单独的CER文件提供给客户端，使用如下命令：

```shell
keytool -keystore /home/wu/keyfile/tomcat.keystore -export -alias tomcat -file /home/wu/keyfile/tomcat.cer
```

#### 导入服务器公钥证书（tomcat.cer）  
由于是自签名的证书，为避免每次都提示不安全。这里双击tomcat.cer安装服务器证书。  
**注意**：将证书填入到“受信任的根证书颁发机构”  
再次重新访问服务器，会发现没有不安全的提示了，同时浏览器地址栏上也有个“锁”图标。  
