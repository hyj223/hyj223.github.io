---
layout: post
title:  "imac 删除项目中svn文件"
date:   2015-11-18 11:29:08 +0800
tags:
    - mac
---

### 问题  
> 今天想把一个项目中的所有.svn的相关文件删除,改为git方式进行管理
可是上传到仓库后发现还有.svn的文件
于是查了查解决方法如何删除svn相关文件
 
**方法如下**  
 1. open terminal
 2. exec 2 command line below  
 
```
    cd yoursvnproject
    find ./ -name ".svn" | xargs rm -Rf
```
