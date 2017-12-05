---
layout:     post
title:      关于Httpd[转载]
subtitle:   httpd的安装,工作模式,模块化特性
date:       2017-12-5
author:     Leshami
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - 集群
    
---
#### 前言
>via Leshami
>原文: http://blog.csdn.net/leshami/article/details/49906229


## 一、httpd的安装

演示环境及版本
    # cat /etc/issue
    CentOS release 6.5 (Final)
    Kernel \r on an \m

    # uname -r
    2.6.32-431.el6.x86_64

查看httpd是否已安装

    # rpm -qa httpd

使用yum列出相关httpd安装文件，此处为本地yum源    

    # yum list |grep httpd     
    httpd.x86_64                 2.2.15-29.el6.centos        local_server 
    httpd-devel.i686             2.2.15-29.el6.centos        local_server 
    httpd-devel.x86_64           2.2.15-29.el6.centos        local_server 
    httpd-manual.noarch          2.2.15-29.el6.centos        local_server 
    httpd-tools.x86_64           2.2.15-29.el6.centos        local_server 

安装及验证httpd  

    # yum -y install httpd
    # rpm -qa |grep httpd
    httpd-tools-2.2.15-29.el6.centos.x86_64
    httpd-2.2.15-29.el6.centos.x86_64

使用rpm方式寻找配置文件   

    # rpm -qc httpd  


常用的配置文件：

    /etc/httpd/conf.d/*.conf        ###辅助配置文件
    /etc/httpd/conf/httpd.conf      ###主配置文件
    /etc/sysconfig/httpd            ###httpd工作模式配置文件

使用rpm方式查看安装位置及生成的二进制文件  

    # rpm -ql httpd  

    主程序：
    /usr/sbin/httpd  MPM模式默认
    /usr/sbin/httpd.event
    /usr/sbin/httpd.worker

使用rpm方式查看包的帮助文件 

    # rpm -qd httpd   

启动脚本：/etc/rc.d/init.d/httpd

日志文件目录：

    /var/log/httpd
        access_log：访问日志
        error_log: 错误日志

站点文档目录：（站点根目录）

    /var/www/html

httpd的工作目录：/var/www

  



## 二、httpd的工作模式

1、MPM： Multipath Processing Module（多路处理模块）

    prefork: 多进程模型，每个进程响应一个请求；稳定性好，但并发能力有限；预先生成多个空闲进程；
        由于prefork使用select()系统调用，所以最大并发不能超过1024；

    worker：多进程模型，每个进程可生成多个线程，每个线程响应一个请求；预先生成多个空闲线程；
    event：一个进程直接响应n个请求；可同时启动多个进程；
        httpd-2.2: 测试使用；    ### Author : Leshami
        httpd-2.4: 可生产使用；  ###  Blog   : http://blog.csdn.net/leshami

2、几种工作方式的切换

prefork模式下      

    # service httpd start
    Starting httpd:                                            [  OK  ]
    # ps -ef|grep httpd |grep -v grep ###一个主进程，生成了8个空闲进程
    root       6413      1  0 09:40 ?        00:00:00 /usr/sbin/httpd
    apache     6416   6413  0 09:40 ?        00:00:00 /usr/sbin/httpd
    apache     6417   6413  0 09:40 ?        00:00:00 /usr/sbin/httpd
    apache     6418   6413  0 09:40 ?        00:00:00 /usr/sbin/httpd
    apache     6419   6413  0 09:40 ?        00:00:00 /usr/sbin/httpd
    apache     6420   6413  0 09:40 ?        00:00:00 /usr/sbin/httpd
    apache     6421   6413  0 09:40 ?        00:00:00 /usr/sbin/httpd
    apache     6422   6413  0 09:40 ?        00:00:00 /usr/sbin/httpd
    apache     6423   6413  0 09:40 ?        00:00:00 /usr/sbin/httpd   

    # ss -tulpn |grep httpd
    tcp    LISTEN     0   128  :::80   :::*      users:(("httpd",6413,4),("httpd",6416,4),("httpd",6417,4),("httpd",6418,4),
    ("httpd",6419,4),("httpd",6420,4),("httpd",6421,4),("httpd",6422,4),("httpd",6423,4)

    # netstat -nltp|grep 80
    tcp        0      0 :::80                       :::*          LISTEN      6413/httpd   

worker工作方式

    # cat /etc/sysconfig/httpd|grep -v ^#
    HTTPD=/usr/sbin/httpd.worker

    # service httpd restart
    Stopping httpd:                                            [  OK  ]
    Starting httpd:                                            [  OK  ]
    [root@orasrv1 ~]# ps -ef|grep httpd|grep -v grep
    root       2261      1  0 17:47 ?        00:00:00 /usr/sbin/httpd.worker
    apache     2264   2261  0 17:47 ?        00:00:00 /usr/sbin/httpd.worker
    apache     2265   2261  0 17:47 ?        00:00:00 /usr/sbin/httpd.worker
    apache     2266   2261  0 17:47 ?        00:00:00 /usr/sbin/httpd.worker

event工作方式

注，尽管2.2版本可以设置工作方式为httpd.event，生产环境不建议使用

    # cat /etc/sysconfig/httpd|grep -v ^#
    HTTPD=/usr/sbin/httpd.event
    [root@orasrv1 ~]# service httpd restart
    Stopping httpd:                                            [  OK  ]
    Starting httpd:                                            [  OK  ]
    [root@orasrv1 ~]# ps -ef|grep httpd|grep -v grep
    root       2402      1  0 17:49 ?        00:00:00 /usr/sbin/httpd.event
    apache     2405   2402  0 17:49 ?        00:00:00 /usr/sbin/httpd.event
    apache     2406   2402  0 17:49 ?        00:00:00 /usr/sbin/httpd.event
    apache     2407   2402  0 17:49 ?        00:00:00 /usr/sbin/httpd.event

  

## 三、httpd模块化特性

高度模块化

        core + modules, 
        DSO: Dynamic Shared Object

模块目录：

        /etc/httpd/modules: 符号链接文件
        /usr/lib64/httpd/modules

模块的查看      
 
    httpd -M          ###查看当前httpd进程的所有模块
    httpd.event -M    ###查看event工作模式下的所有模块 更正@20160712
    httpd.worker -M   ###worker工作模式下的所有模块  更正@20160712
    httpd.worker -l   ###worker工作模式下的核心模块  更正@20160712

模块的查看示例 

    # httpd -M
    Loaded Modules:
     core_module (static)
     mpm_prefork_module (static)
     http_module (static)
     so_module (static)
     auth_basic_module (shared)
      ..............

    # httpd.event -l
    Compiled in modules:
      core.c
      event.c
      http_core.c
      mod_so.c

模块的动态装载与卸载

    # cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
    # cat /etc/httpd/conf/httpd.conf |grep authn_alias_module
    LoadModule authn_alias_module modules/mod_authn_alias.so
    # vi /etc/httpd/conf/httpd.conf  ###注释mod_authn_alias.so模块
    # cat /etc/httpd/conf/httpd.conf |grep authn_alias_module
      #LoadModule authn_alias_module modules/mod_authn_alias.so
    # service httpd restart
    # httpd -M   ###使用该方式前后进行对比即可知道模块是否装载或卸载    

   


## 四、验证httpd服务


# echo "<h1>orasrv1.xlk.com</h1>" >/var/www/html/index.html
# curl http://192.168.21.10
<h1>orasrv1.xlk.com</h1>