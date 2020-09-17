---
title: Ubuntu/CentOS搭建NFS服务
date: 2020-04-16 16:04:17
categories: Linux
tags:
- Linux
- Ubuntu
---

## 1、NFS 介绍

NFS 即网络文件系统（Network File-System），可以通过网络让不同机器、不同系统之间可以实现文件共享。通过 NFS，可以访问远程共享目录，就像访问本地磁盘一样。NFS 只是一种文件系统，本身并没有传输功能，是基于 RPC（远程过程调用）协议实现的，采用 C/S 架构。

## 2、安装 NFS 软件包

Ubuntu：

```bash
apt-get install nfs-kernel-server  # 安装 NFS服务器端
apt-get install nfs-common rpcbind # 安装 NFS客户端
```

CentOS：

```bash
yum install -y nfs-common nfs-utils rpcbind
```

## 3、添加 NFS 共享目录

若需要把 `/nfsroot` 目录设置为 NFS 共享目录，请在`/etc/export`文件末尾添加下面的一行：

`/nfsroot *(rw,no_root_squash,no_all_squash,sync) # * 表示允许任何网段 IP 的系统访问该 NFS 目录`

新建`/nfsroot`目录，并为该目录设置最宽松的权限：

```bash
mkdir /nfsroot
chmod 777 /nfsroot
```

## 4、启动 NFS 服务

`/etc/init.d/nfs-kernel-server start`

在 NFS 服务已经启动的情况下，如果修改了 `/etc/exports` 文件，需要重启 NFS 服务，以刷新 NFS 的共享目录。

`/etc/init.d/nfs-kernel-server restart`

## 5、测试 NFS 服务器

`sudo mount -t nfs <ip_addr>:/nfsroot /mnt -o nolock`

`<ip_addr>`为主机 ip，`/nfsroot`为主机共享目录，`/mnt` 为设备挂载目录，如果指令运行没有出错，则 NFS 挂载成功，在主机的`/mnt` 目录下应该可以看到`/nfsroot`目录下的内容（可先在 nfsroot 目录下新建测试目录），如需卸载使用

`umount /mnt`
