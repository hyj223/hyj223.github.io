---
layout:     post
title:      "n-tree-impl"
description:   ""
date:       2015-12-15
tags:
    - 算法

---  

### 分析

 * 用三种颜色来标记节点的选中状态（green-该节点实际被选中，yellow-该节点未被选中，但子级节点又被选中，white-该节点未被选中） 
 * 当选中一个节点时：  
    * 对应的上级节点根据其下一级节点的选中状态变化。  
    若上级节点的下一级节点全部选中，则上级节点为全选状态，若上级节点的下一级节点部分选中，则上级节点为部分选中
    * 对应的下级节点改为全选中状态。  
 
 * 当取消某个节点的选中时：
    * 对应的上级节点根据其下一级节点的改变而改变。  
     若上级节点的下一级节点部分选中，则上级节点为部分选中，若上级节点的下一级节点都未选中，则上级节点为未被选中状态~
    * 其下级节点则改为未选中状态。

恩，就是酱样子。

