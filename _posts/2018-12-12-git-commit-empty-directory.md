---
layout: post
title: Git不能提交空目录原因和解决方法
category: Linux运维
---

## Git不能提交空目录原因和解决方法

### 问题背景

在做资源库由svn迁移到gitlab时发现，很多目录提交到gitlab，即使`git add .`， 小伙伴们`check out`也没有这些目录。

### Git不能提交空目录原因

原因是这些目录都是空目录，git和svn不同，git仅仅跟踪文件的变动，不跟踪目录，git会忽略提交空目录。

### Git不能提交空目录解决方法

解决方法：在空目录中新建一个空文件，比如**readme.md**，**.gitkeep**

快速在本地查找到项目中的空目录并新建空文件

```cmd
find . -type d -empty -exec touch {}/.gitkeep \;
```
做完后，再`git add .`就会发现这些空目录里包含了**.gitkeep**，就可以`git commit`和`git push`了。