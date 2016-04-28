---
title: git补充(常见命令)
tag: git
---
## {{ page.title }}

想要了解git某个命令的基本常见操作查阅对应命令的example即可,比如想查阅git add的常见
操作,执行 `git add --help` 然后拉到最下面看example,里面有常见的操作.本文还是将
常见的操作列举出来.

<br/>

### 创建新版本库

拷贝已有的版本库

~~~
git clone <url>
~~~

初始化新的版本库(将本地的版本作为初始版本,不能pull,因为本地已经有写好的代码)

~~~
1.  git init
2.  在github创建一个新的repository,并获取其url,
3.  git remote add origin <url> #<url>是刚刚获取的url
4.  git push -f origin master
~~~

<br/>

### 切换分支

~~~
git checkout dev #切换当前分支到dev
git checkout -b dev
#将dev作为当前分支的拷贝并切换至dev
git checkout -b dev origin/dev
#dev 作为远程dev分支的拷贝并切换至dev
~~~

<br/>

### 分支操作

~~~
git branch #查看本地分支
git branch -r #查看远程分支
git branch -a #查看所有分支(本地加远程)
git branch -d dev #删除本地dev分支
git push origin :dev #删除远程分支(也就是push一个空的分支过去)
~~~

<br/>

### 远程代号
远程代号一个字符串,这个字符串指的是一个url地址,比如origin.

~~~
git remote -v #列出所有代号
git remote add origin <url> #让origin指向<url>
git remote remove origin #删除远程代号origin
~~~

<br/>

### 补充修改和提交

~~~
git status #非常常用,查看workspace还没有提交到index的内容
            #以及index还没有提交到local repository的内容

git diff  #比较workspace和index的不同
git diff . #同上,比较范围是当前目录
git diff <filename> #同上,比较范围是指定文件
git diff HEAD  #比较workspace和local repository的不同
git diff HEAD .
git diff HEAD <filename>
git diff --cached #比较index和local repository的不同
git diff --cached .
git diff --cached <filename>

git rm <filename> #同时删除workspace和index中的<filename>对应的文件
git rm --cached <filename> #只删除index里面的文件

git cherry-pick <commit> #相当于选择<commit>作为一个新的
#commit提交到当前的branch上面
~~~

### 合并分支

~~~
git merge dev #将dev分支merge到当前的分支中
git rebase -i <commit>
#找到当前分支和commit第一个不同的commit1,然后从commit1开始
#把当前的所有commit放在<commit>上面
~~~

<br/>

### 其它常用

~~~
git blame <filename> #查看文件内容每一行的作者,修改日期等
git log #从当前分支的最后一个commit开始往前显示每一个commit的信息
git log -n #n是数字,含义同上,但是只显示最近的n个commit信息
git log -p <filename> #查看某个文件的提交历史
~~~