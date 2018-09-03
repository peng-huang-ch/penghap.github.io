---
title: FTP挂载
date: 2018-01-14 19:34:40
tags: [服务器, linux]
categories: 文件挂载
---
### 一 [介绍](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)

&#160; &#160; &#160; &#160; ftp（File Transfer Protocol）是用于在网络上进行文件传输的一套标准协议，使用客户/服务器模式。它属于网络传输协议的应用层。

服务端使用vsftpd搭建ftp服务器
客户端使用curlftpfs实现挂载


### 二 FTP Server

#### 2.1 安装
使用yum 安装 vsftpd

```sh
sudo yum install vsftpd
```

#### 2.2 测试
安装FTP 客户端

```sh
sudo yum install ftp
```

    ftp hap@192.168.1.22

### 三 FTP client

#### 3.1 安装 curlftpfs

使用yum 安装 curlftpfs

```sh
sudo yum install curlftpfs
```

#### 3.2 挂载测试

    sudo curlftpfs -o  ftp://username:password@192.168.1.22 /home/hap/share

