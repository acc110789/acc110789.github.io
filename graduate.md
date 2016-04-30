---
title: 毕业设计材料
layout: post
---
## {{page.title}}

<br/>

### 前言部分

* 目前,国内飞机(我坐过的)还没有配备舱内娱乐设施,但是据说某些国际航班
已经配备了舱内娱乐设施,着眼于未来更好的舱内娱乐体验,需要开展定向天线
技术的研究.


<br/>

### 正文部分

* 首先介绍自己写的仿真软件,软件的结构等稍微介绍一下.
* 然后在全向天线部分与已经存在的smpl比较一下.
* 然后做一些定向天线方面的试验来验证软件的正确性.
* 可能还需要有一小部分的idea.

<br/>

### 可能面临的诘问

1. 问:如果飞机上已经有wifi了,为什么还要配备定线天线,在飞机上搞定向天线技术的
意义是什么?\\
答:围绕带宽和能耗说话,使用定向天线技术能很大程度上缓解无线传感器的能耗问题.至于
带宽问题,其实3G技术已经有200KB/s的传输速度,其实有200KB/s基本能满足上网需求了
所以有为什么还要搞4G技术的,OK,因为还不能看高清视频甚至是极清视频.所以又出现了4G
技术,但是现在又在开展5G技术的研究,既然已经有4G可以满足需求了,为什么还要开展5G技术
的研究呢?因为人们总是更多更大量的信息需要传递,比如又出现了360全景视频,VR等对带宽
需求更大的应用场景,应该说这些需求原本就一直存在,这也就是驱使带宽不断变大的一个动力?
2. 问:什么是多径效应呢?
3. 问:定向天线路由器有多大呢?\\
答:![802.11ad](http://img1.cache.netease.com/catchpic/E/E0/E0FE383DCC3C6E73BB5754D850BC5548.jpg)
4. mac层的问题真的没有解决吗,为什么产品都出来了?

<br/>

### 目前想到的可能的idea
首先,60G定线向比较与全向天线技术能提高带宽的原因有两个.

1. 本来定线天线的带宽就比全向天线的带宽更高.
2. 定向天线技术能实现空间复用.

<br/>

如果只单纯利用第一点,即把定线天线当全向天线用,有一种idea是rts和cts挨着每个扇区都
发送一次,但是做试验的过程中发现一个问题,由于rts比SLOT长太多,所以仍然极其容易发生碰撞.
所以把整个过程重新思考一下,我们只是为了让周围的节点知道我要进行通信了,你们赶紧设置NAV吧
所以可以尝试重新定义一种新的桢类型,这种桢只有NAV,没有src和target的地址.

