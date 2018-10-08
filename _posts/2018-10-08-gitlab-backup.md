---
layout: post
title: Gitlab数据备份和还原
category: Linux运维
---


## gitlab数据备份和还原

### 数据备份

默认备份文件储存在目录`/var/opt/gitlab/backups`
比如：
```
# 1536775227_2018_09_13是时间戳，11.2.1是gitlab版本号，这两个组合成用于还原时的编号
/var/opt/gitlab/backups/1536775227_2018_09_13_11.2.1_gitlab_backup.tar
```

另外建议备份:

```
# 配置文件
/etc/gitlab/gitlab.rb
# nginx配置文件
/var/opt/gitlab/nginx/conf
# 邮件配置
/etc/postfix/main.cfpostfix
```

备份命令：
```shell
sudo gitlab-rake gitlab:backup:create
```
备份到此也就完成了，但是如果不想每次都去敲命令，也可以配置定时任务，让系统自动备份gitlab，方法如下:
```shell
sudo crontab -e
```
添加以下内容，并保存
```shell
# 每日2点备份gitlab
0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1
```

注意：环境变量CRON=1的作用是如果没有任何错误发生时，抑制备份脚本的所有进度输出  

或者直接编辑/etc/crontab 文件，即`sudo vi /etc/crontab`，然后添加相应的任务

> crontab小知识  
> 分 时 日 月 周 命令  
> 上面的解释就是每日2时0分，执行命令/opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1  


备份文件如果嫌多了，可以设置只保存最近7天的备份  
编辑 `/etc/gitlab/gitlab.rb` 配置文件,找到如下代码,删除注释 #  保存 
```shell
gitlab_rails['backup_keep_time'] = 604800
```
重新加载gitlab配置文件
```shell
sudo gitlab-ctl reconfigure
```

### 数据还原
gitlab从备份中还原

Gitlab的恢复操作会先将当前所有的数据清空，然后再根据备份数据进行恢复
```shell
#将备份文件拷贝到备份目录
sudo cp 1536775227_2018_09_13_11.2.1_gitlab_backup.tar /var/opt/gitlab/backups/ 
```

停止相关数据连接服务
```shell
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq
```

从备份恢复(备份文件名的时间戳版本前缀)
```shell
sudo gitlab-rake gitlab:backup:restore BACKUP=1536775227_2018_09_13_11.2.1
```
从默认备份恢复（backups目录下只有一个备份文件时）：`sudo gitlab-rake gitlab:backup:restore`

启动Gitlab
```shell
sudo gitlab-ctl start
```

恢复命令完成后，可以check检查一下恢复情况
```shell
sudo gitlab-rake gitlab:check SANITIZE=true
```
### 数据迁移
Gitlab迁移与恢复一样，但是要求两个GitLab版本号一致


