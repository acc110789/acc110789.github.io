---
title: git图解基本概念
tag: git
---
## {{page.title}}

![git详解](http://i.stack.imgur.com/caci5.png)

在看git文档的时候,概念要先搞清楚:\\
图中的workspace也叫working tree,也叫工作区.\\
index也叫暂存区,还叫staged index,staged area.staging area\\
local repository也叫本地仓库(翻译过来).\\
上面的图已经是比较清楚的展示了git的主要操作了,下面是具体解释.

### workspace ---> index
将index更新到workspace当前的状态\\
不论是新建的文件还是被修改过的文件,都要执行add才能从workspace添加到index区域.

~~~
git add <filename1> <filename>...
git add .
~~~

&lt;filename&gt;是指文件的名称,后面遇到&lt;filename&gt;同理也是指文件名称,\\
"."点一般是指当前目录下所有文件"前者是添加指定的文件,后者是添加当前目录下所有文件.

### index ---> workspace
把workspace当前的状态reset到index的当前状态

~~~
git checkout . #将当前目录下的所有文件reset
git checkout <filename1> <filename>...  #reset指定的文件
~~~

此命令仅仅把index中有的东西reset回来,比如workspace在add到index中后又对某个文件
做了更改,这个时候就会把workspace的这个文件reset到index的版本.如果add到index之后又
把这个文件删除了,也能通过reset restore回来.但是如果在workspace中新建一个文件,这个文件
没有被add到index,这个时候执行reset,这个文件是不会消失的.所以需要搞清楚这个含义:把
index中有的东西reset回来,而workspace中新增的文件是不会因为reset被删除的.

### index ---> local repository
将index的当前状态更新到local repository中

~~~
git commit -m <message>
~~~

### local repository ---> index
将index的状态reset到local repository当前的状态\\
暂时没有发现专门实现功能的命令,reset有一个参数 --mixed,除了将HEAD reset到
相应的commit,还有个side effect是将index reset到对应的commit的状态

~~~
git reset --mixed <commit>
~~~

### workspace ---> index ---> local repository
将workspace的更改更新到index和local repository中

~~~
git commit -a -m <commit message>
~~~

-a 参数\\
by using the -a switch with the commit command to
automatically "add" changes from all known files (i.e. all files
that are already listed in the index) and to automatically "rm"
files in the index that have been removed from the
working tree, and then perform the actual commit;

### local repository ---> index ---> workspace
将index和workspace都reset到local repository当前的状态

~~~
git checkout HEAD .
git checkout HEAD <filename>
git reset --hard HEAD
~~~

这里又遇到了这个`git checkout`,为什么有时候是更换branch,而这里又是reset的作用?
checkout的解释是这样的.\\
Updates files in the working tree to match the version
in the index or the specified tree. If no paths are given, git checkout
 will also update HEAD to set the specified branch as the current branch.\\
 所以,当有路径(path)的时候,是不会更新HEAD的,"."和&lt;filename&gt;都是路径.当没有
 路径的时候,就要更新HEAD,也就是更换branch了.

### diff(workspace,index)
比较workspace和index之间的不同

~~~
git diff
git diff .
git diff <filename>
~~~

### diff(workspace,local repository)
比较workspace和local repository之间的不同

~~~
git diff HEAD
git diff HEAD .
git diff HEAD <filename>
~~~

有一点需要注意(图中并没有予以说明):

> index和local reposiry里面的东西都是放到objects里面的.







