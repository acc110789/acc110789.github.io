---
layout: post
title: git总结
category: 技术
tag: git
---
## {{page.title}}

![git详解](http://i.stack.imgur.com/caci5.png)

这张图已经是比较清楚了,稍微解释一下图.<br/>
几个概念:<br/>
图中的workspace有时候也叫working tree,也叫工作区.<br/>
index也叫暂存区,还叫staged index,staged area.<br/>
local repository也叫本地仓库(翻译过来)吧.

有一点需要注意(图中并没有予以说明):
> index和local reposiry里面的东西都是放到objects里面的.

### workspace ---> index<br/>
将index更新到workspace当前的状态<br/>
不论是新建的文件还是被修改过的文件,都要执行add才能从workspace添加到index区域.<br/>
命令: `git add <filename><br/>` 或者 `git add .<br/>`
说明:<filename>是指文件的名称,后面遇到<filename>同理也是指文件名称,"."点一般是指所有文件"<br/>
前者是添加一个文件,后者是添加所有文件.

### index ---> workspace<br/>
把workspace当前的状态reset到index的当前状态<br/>
命令: `git checkout .` 或者 `git checkout <filename>`

### index ---> local repository<br/>
将index的当前状态更新到local repository中<br/>
命令:`git commit -m <message>`

### local repository ---> index<br/>
将index的状态reset到local repository当前的状态<br/>
命令:

### workspace ---> index ---> local repository <br/>
将workspace的更改更新到index和local repository中<br/>
命令: `git commit -a`

### local repository ---> index ---> workspace<br/>
将index和workspace都reset到local repository当前的状态<br/>
命令: `git checkout HEAD .` 或者 `git checkout HEAD <filename>`

### diff(workspace,index)<br/>
比较workspace和index之间的不同<br/>
命令:`diff`

### diff(workspace,local repository)<br/>
比较workspace和local repository之间的不同<br/>
命令:`diff HEAD`







