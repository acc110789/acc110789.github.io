---
layout: post
title: git总结
category: 技术
tag: git
---
##{{page.title}}

![git详解](http://i.stack.imgur.com/caci5.png)

这张图已经是比较清楚了,稍微解释一下图.<br/>
几个概念:<br/>
图中的workspace有时候也叫working tree,也叫工作区.<br/>
index也叫暂存区,还叫staged index,staged area.<br/>
local repository也叫本地仓库(翻译过来)吧.

有一点需要注意(图中并没有予以说明):
> index和local reposiry里面的东西都是放到objects里面的.

workspace --> index : git add <filename><br/> 或者 git add .<br/>
前者是添加一个文件,后者是添加所有文件
<filename>是指文件的名称,后面遇到<filename>同理也是指文件名称,不论是新建的
 文件还是被修改过的文件,都要执行add才能从workspace添加到index区域.

index ---> workspace : git checkout . 或者 git checkout <filename><br/>
前者把整个workspace当前的状态reset到index的当前状态,后者是把workspace中
一个文件reset到index的状态.

index ---> local repository: git commit -m <message><br/>
这个就不解释了.

local repository ---> index ---> workspace

http://stackoverflow.com/questions/3689838/difference-between-head-working-tree-index-in-git

http://blog.csdn.net/felix_f/article/details/8777463

明天再写




