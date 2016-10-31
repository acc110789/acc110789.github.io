---
title: mysql在osx上面的配置过程
tag: 数据库
---

## {{ page.title }}

### 第一个,安装
终端下面执行如下的命令

~~~
sudo brew install mysql
~~~

在实验室和宿舍下载的很慢,后来直接开vpn,秒下载完成。

### 第二步,修改权限
没有这一步的话,在后面的配置过程经常会遇到各种问题,打开log文件就会发现全是权限问题。索性
一开始就把权限全部改了。

~~~
cd /usr/local/var/mysql
su
chmod -R 777 .
~~~

### 第三步,更改root的密码

~~~
sudo mysqld_safe --skip-grant-tables    #应该开一个安全模式的mysql服务的意思吧
mysql -u root        #然后在另一个终端下就可以不用密码直接登录进去了
~~~

进去mysql之后执行

~~~
select * from mysql.user where user='root';
~~~

可以看到`user`表的结构。下面是一个截图。

<img width="700dp"  src="/images/user表结构.png"/>

可以看到字段的最后一行的第一项`authentication_string`,这个就是密码,不过可以看到这个是一个散列值。
所以mysql存储的应该不是密码,而是密码的散列值。下面给出更改密码的语句。下面的语句中假设我想把`root`这个用户的
密码变成"1234567890"。

~~~
UPDATE mysql.user SET authentication_string=PASSWORD('1234567890') WHERE user='root';
~~~

再执行一次

~~~
select * from mysql.user where user='root';
~~~

观察`authentication_string`是不是和之前不一样了。

然后再执行

~~~
FLUSH PRIVILEGES;
~~~

这样下次再进来就要输入密码了。然后执行`\q`退出mysql

### 第四步,就正常使用了

~~~
#先把之前的mysqld_safe关掉吧,这个好想有点坑,kill -15 ,ctrl + c都关不掉,要强行kill -9 才行啊
sudo mysql.server start  #服务端开启服务
mysql -h localhost -u root -p #客户端这边使用密码登录即可
~~~

### 补充

第四步之后能进去了,但是没法做事情,总是提示让reset密码。执行下面的命令

~~~
SET PASSWORD = PASSWORD(‘your new password‘);
ALTER USER root@localhost PASSWORD EXPIRE NEVER;
flush privileges;
~~~

然后应该就可以了。