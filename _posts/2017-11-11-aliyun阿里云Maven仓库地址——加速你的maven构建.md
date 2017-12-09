---
layout:     post
title:      aliyun阿里云Maven仓库地址——加速你的maven构建
subtitle:   Maven仓库
date:       2017-12-5
author:     Corliss
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - 集群
    
---

#### 前言
>maven仓库用过的人都知道，国内有多么的悲催。还好有比较好用的镜像可以使用，尽快记录下来。速度提升100倍。
>http://maven.aliyun.com/nexus/#view-repositories;public~browsestorage

在maven的settings.xml 文件里配置mirrors的子节点，添加如下mirror



    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror> 