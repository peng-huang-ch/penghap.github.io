---
title: sshfs
date: 2018-01-14 19:55:40
tags: [服务器, linux]
categories: 文件挂载
---
### 一 介绍 

&#160; &#160; &#160; &#160; SSHFS（SSH Filesystem）是一种通过普通ssh连接来挂载和与远程服务器或工作站上的目录和文件交互的文件系统客户端。该种客户端通过SSH文件传输协议（SFTP）与远程文件系统交互，这是一种通过任何可靠数据流提供文件访问、文件传输和文件管理功能的网络协议，它在设计上是Secure Shell（SSH）协议2.0版的一个扩展。通过它远程主机的配置无需作任何改变，就可以透过 SSH 协议来挂载远程文件系统了，非常方便及安全。

[wiki](https://zh.wikipedia.org/wiki/SSHFS) 
[github](https://github.com/libfuse/sshfs)

### 二 安装（Centos7）
使用yum 安装

```
yum install -y epel-release 
yum install -y fuse-sshfs
```

### 三 使用

#### 3.1 创建挂载目录
```
mkdir /home/hap/mnt/data
```

#### 3.2 使用sshfs挂载
```
sshfs {{user}}@{{server}}:{{desiredremote share}} {{desired local mount point}} -o allow_other 
```
如：

    sshfs hap@192.168.1.22:/home/hap/share /home/test/share -o allow_other,nonempty,auto_cache,reconnect,IdentityFile=~/.ssh/id_rsa


#### 3.3 卸载挂载目录
    fusermount -u mount_point (如果提示 device is busy )
    sudo umount -fl mount_point

#### 3.4 参数说明
-o reconnect 自动重连
-o nonempty 允许挂载目录非空
-o allow_other 挂载过来的目录非root能够访问
-o reconnect 断网自动重连

下面是一些具体介绍 [来源](http://blog.csdn.net/netwalk/article/details/12952719)
```
general options:
    -o opt,[opt...]        mount options
    -h   --help            print help
    -V   --version         print version

SSHFS options:
    -p PORT                equivalent to '-o port=PORT'
    -C                     equivalent to '-o compression=yes' #启用压缩,建议配上
    -F ssh_configfile      specifies alternative ssh configuration file #使用非默认的ssh配置文件
    -1                     equivalent to '-o ssh_protocol=1' #不要用啊
    -o reconnect           reconnect to server               #自动重连
    -o delay_connect       delay connection to server
    -o sshfs_sync          synchronous writes
    -o no_readahead        synchronous reads (no speculative readahead) #提前预读
    -o sshfs_debug         print some debugging information
    -o cache=BOOL          enable caching {yes,no} (default: yes) #能缓存目录结构之类的信息
    -o cache_timeout=N     sets timeout for caches in seconds (default: 20)
    -o cache_X_timeout=N   sets timeout for {stat,dir,link} cache
    -o workaround=LIST     colon separated list of workarounds
             none             no workarounds enabled
             all              all workarounds enabled
             [no]rename       fix renaming to existing file (default: off)
             [no]nodelaysrv   set nodelay tcp flag in sshd (default: off)
             [no]truncate     fix truncate for old servers (default: off)
             [no]buflimit     fix buffer fillup bug in server (default: on)
    -o idmap=TYPE          user/group ID mapping, possible types are:  #文件权限uid/gid映射关系
             none             no translation of the ID space (default)
             user             only translate UID of connecting user
    -o ssh_command=CMD     execute CMD instead of 'ssh'
    -o ssh_protocol=N      ssh protocol to use (default: 2) #肯定要2的
    -o sftp_server=SERV    path to sftp server or subsystem (default: sftp)
    -o directport=PORT     directly connect to PORT bypassing ssh
    -o transform_symlinks  transform absolute symlinks to relative
    -o follow_symlinks     follow symlinks on the server
    -o no_check_root       don't check for existence of 'dir' on server
    -o password_stdin      read password from stdin (only for pam_mount)
    -o SSHOPT=VAL          ssh options (see man ssh_config)

Module options:

[subdir]
    -o subdir=DIR       prepend this directory to all paths (mandatory)
    -o [no]rellinks     transform absolute symlinks to relative

[iconv]
    #字符集转换,对我这种UTF8控,默认已经是最好的
    -o from_code=CHARSET   original encoding of file names (default: UTF-8)
    -o to_code=CHARSET      new encoding of the file names (default: UTF-8)
```

### 四 开机自动挂载
1. 配置好ssh免秘钥登录
2. 修改 /etc/fstab，添加挂载命令

### 五 参考
[1] [sshfs说明](https://zh.wikipedia.org/wiki/SSHFS)
[2] [Ubuntu下使用sshfs挂载远程目录到本地](http://blog.csdn.net/netwalk/article/details/12952719)
