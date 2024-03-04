# linux虚拟机无法通过主机名访问hadoop集群web端的解决办法

## 目录

[TOC]

## 问题描述

在Linux搭建完hadoop集群环境时，访问hdfs时用主机名+端口号无法访问，只能通过ip地址访问，这通常是主机名与ip地址的映射除了问题，在网上搜索答案时，通常只有几种基础的解决办法，我在操作完后仍然不行，又通过一些资料的查找找到了解决办法。

## 解决方案

* 关闭linux虚拟机的防火墙

  ```shell
  sudo systemctl stop firewalld
  ```

* 关闭防火墙开机自启

  ```shell
  sudo systemctl disable firewalld
  ```

* 检查Linux中的/etc/hosts文件，查看主机名与IP地址时候映射正确

  ```shell
  vim /etc/hosts
  ```

  添加如下信息（ip地址 主机名）

  ```
  192.168.10.102 hadoop102
  192.168.10.103 hadoop103
  192.168.10.104 hadoop104
  ```

* 修改linux中的workers

  ```shell
  vim /opt/module/hadoop-3.1.3/etc/hadoop/workers
  ```

  此处以虚拟机的hadoop地址为准

  添加hadoop集群主机名

  ```shell
  hadoop102
  hadoop103
  hadoop104
  ```

* 修改windows本地的hosts文件夹

  这个文件在`C:\Windows\System32\drivers\etc\hosts`，添加如下信息

  ​	<img src="D:\App\Typora\photoData\image-20240207103853935.png" style="zoom:67%;" />

## 注意（特殊情况）

完成上述操作基本可以实现通过主机名访问，而在网络上搜索此问题给出的答案也是这样，如果完成上面全部操作仍然不能通过主机名访问，大概率是windows的hosts文件编码的问题，对于 Windows 系统来说，当您修改了 hosts 文件后，有可能是编码的原因导致修改不生效：

### hosts文件格式问题

* 在修改hosts文件时，不要使用notepad++等软件，使用记事本编辑

<img src="D:\App\Typora\photoData\image-20240207104722186.png" style="zoom:50%;" />

* 添加完信息后，另存到桌面，文件格式选择ASNI

<img src="D:\App\Typora\photoData\image-20240207104850779.png" style="zoom:50%;" />

* 保存之后文件名会多出后缀`.txt`，打开文件发现有乱码（UTF-8空格变成了乱码），如果有乱码去掉乱码，重命名去掉后缀，然后复制替换原来的hosts文件

* 如果没有及时生效，刷新一下DNS即可，cmd执行以下命令

  ```
  ipconfig /flushdns
  ```

  