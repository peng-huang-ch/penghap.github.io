---
title: NFS
date: 2018-01-14 19:45:40
tags: [服务器, linux]
categories: 文件挂载
---
### 一 介绍 

&#160; &#160; NFS（Network File System）即网络文件系统，是一种分布式文件系统协议。其功能旨在允许客户端主机可以像访问本地存储一样通过网络访问服务器端文件。
&#160; &#160; NFS是基于RPC服务，NFS的需要端口都需要向RPC注册。

#### 1.1 NFS的工作流程
1. NFS客户端发起存取文件的请求，客户端本地的RPC(rpcbind)服务会通过网络向NFS服务端的RPC的111端口发出文件存取功能的请求
2. NFS服务端的RPC找到对应已注册的NFS端口，通知客户端RPC服务
3. 客户端获取正确的端口，并与NFS daemon联机存取数据
4. 存取数据成功后，返回前端访问程序，完成一次存取操作

#### 1.2 几个服务介绍
下面主要介绍的主要是NFS V4，在V4中rpcbind ，lockd，pc.statd已经不在需要，但是rpc.mountd还是需要的，它不涉及中间的读写，只是确保能否访问。

| 服务 | 端口号 | 必须 | 描述 |
| ---- | -------- | ----- | ----- |
| rpc | 111 | 是 | RPC服务，NFS需要 |
| nfs | 2409 | 是 | NFS服务，文件共享需要 |
| rpc.mountd | 随机 | 是 | NFS服务，文件共享需要 |
| rpcbind | 随机 | V4可以不要 | 允许NFS客户端访问 |
| nfslock | 随机 | V4可以不要 | 允许NFS客户端锁定服务器上的文件 |
| rpc.nfsd | 随机 | V4可以不要 | 服务器通告明确的NFS版本和协议，以满足NFS客户端的动态需求 |

[详细文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-nfs)

### 二 安装（Centos7）

2.1 使用yum 安装
```sh
yum install  rpcbind nfs-utils
yum install  rpcbind rpcbind
```

2.2 设置开机自启动
```sh
systemctl enable rpcbind
systemctl enable nfs-server
systemctl start rpcbind
systemctl start nfs-server
```

### 三 NFS Server
#### 3.1 创建挂载的目录
```
mkdir /home/hap/share
```

#### 3.2 添加可以导出目录
```sh
/home/hap/share 192.168.1.123(rw,sync,no_root_squash)
```
#### 3.3 重新读取并应用配置文件
```sh
sudo exportfs -r
```

### 四 NFS client
#### 4.1 显示可以挂载的目录
```sh
showmount -e 192.168.1.22 #服务器地址
mount -t nfs 192.168.1.22:/home/hap/share /home/test/share
```

#### 4.2 卸载挂载目录
    fusermount -u mount_point (如果提示 device is busy )
    sudo umount -fl mount_point

### 五 防火墙设置
NFS中 mountd, rquotad, nlockmgr是使用的随机端口，这样对于我们开防火墙就不是很方便了，所以我们需要固定下他们使用的端口号。

#### 5.1 两（三）个端口号
  在V4版本中，一些服务已经不再需要了，所以这里只是使用了rpc，nfs，rpc.mountd三个服务。
##### 5.1.1 /etc/sysconfig/nfs的配置文件
  固定 mountd
```sh
# Optional arguments passed to rpc.mountd. See rpc.mountd(8)
RPCMOUNTDOPTS=""
# Port rpc.mountd should listen on.
MOUNTD_PORT=111
#
```
我这里使用的111端口，如果不行就换成892

##### 5.1.2 端口号使用情况
| 服务 | 端口号 | tcp/udp |
| ---- | -------- |
| rpc | 111 | 是 | tcp 和 udp |
| nfs | 2409 | 是 | tcp 和 udp |
| rpc.mountd |  111 或 892  | tcp 和 udp |

#### 5.2 五个端口号
  固定端口nfs 2049、portmapper111 ，另外3个服务端口可设置为mountd 892、rpc.statd 662、 nlockmgr 32803/tcp、32769/udp

##### 5.2.1 /etc/sysconfig/nfs的配置文件
  固定 mountd, rpc.statd

```sh
# Optional arguments passed to rpc.mountd. See rpc.mountd(8)
RPCMOUNTDOPTS=""
# Port rpc.mountd should listen on.
MOUNTD_PORT=892
#
# Optional arguments passed to rpc.statd. See rpc.statd(8)
STATDARG=""
# Port rpc.statd should listen on.
STATD_PORT=662
# Outgoing port statd should used. The default is port
# is random
STATD_OUTGOING_PORT=2020
# Specify callout program
#STATD_HA_CALLOUT="/usr/local/bin/foo"
#
# Optional arguments passed to sm-notify. See sm-notify(8)
SMNOTIFYARGS=""
#
# Optional arguments passed to rpc.idmapd. See rpc.idmapd(8)
RPCIDMAPDARGS=""
```

##### 5.2.2 /etc/modprobe.d/lockd.conf 的配置文件

    固定 nlockmgr

```sh
# Set the NFS lock manager grace period. n is measured in seconds. 
#options lockd nlm_grace_period=90
#
# Set the TCP port that the NFS lock manager should use. 
# port must be a valid TCP port value (1-65535).
options lockd nlm_tcpport=32803
#
# Set the UDP port that the NFS lock manager should use.
# port must be a valid UDP port value (1-65535).
options lockd nlm_udpport=32769
#
# Set the maximum number of outstanding connections 
#options lockd nlm_max_connections=1024
```
##### 5.2.3 端口号使用情况
| 服务 | 端口号 | tcp/udp |
| ---- | ------ | -------- |
| rpc | 111 | tcp 和 udp |
| nfs | 2409 | tcp 和 udp |
| rpc.statd | 662 | tcp 和 udp |
| rpc.mountd |  892 | tcp 和 udp |
| nfslock | 32803 | tcp |
| nfslock | 32769 | udp | 

这里的32803和32769可以合并为一个端口，如32803(tcp 和 udp)

### 六 挂载
参数说明

| 参数 | 说明 | 
| ---- | -------- |
| ro  | 只读访问 | 
| rw  | 读写访问|
| sync | 所有数据在请求时写入共享 |
| async |  先暂存于内存当中，而非直接写入硬盘 |
| secure |  NFS通过1024以下的安全TCP/IP端口发送 |
| insecure  | NFS通过1024以上的端口发送 | 
| hide| 在NFS共享目录中不共享其子目录  | 
| no_hide | 共享NFS目录的子目录 |
| subtree_check | 强制NFS检查父目录的权限（默认 |
| no_subtree_check | 和上面相对，不检查父目录权限 
| all_squash | 共享文件的UID和GID映射匿名用户anonymous，适合公用目录。 
| no_all_squash | 保留共享文件的UID和GID（默认） 
| root_squash |root用户的所有请求映射成如anonymous用户一样的权限（默认） 
| no_root_squas | root用户具有根目录的完全管理访问权限 
| anonuid=xxx | 指定NFS服务器/etc/passwd文件中匿名用户的UID 

挂载测试
```
showmount -e 192.168.1.22
mount -t nfs 192.168.1.22:/home/hap/share /home/hap/share
```
查看挂载情况

    df -h 

### 参考
[1] [Centons nfs介绍](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-nfs)
[2] [Centons nfs配置介绍](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/nfs-serverconfig)
[3] [鸟哥 NFS服务介绍](http://linux.vbird.org/linux_server/0330nfs.php)
[4] [CentOS7安装配置 NFS](http://wangshengzhuang.com/2017/06/07/Linux/Linux%E5%9F%BA%E7%A1%80/CentOS%207%20%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE%20NFS/)
