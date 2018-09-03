---
title: SFTP和SSH分离
date: 2018-01-14 20:55:40
tags: [服务器, linux, ssh]
categories: 文件挂载
---
### 一 [介绍](https://zh.wikipedia.org/wiki/SSH%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)
#### 1.1 描述
&#160; &#160; SFTP（SSH File Transfer Protocol）即安全文件传送协议），是一数据流连接，提供文件访问、传输和管理功能的网络传输协议。其功能旨在允许客户端主机可以像访问本地存储一样通过网络访问服务器端文件。

&#160; &#160;sftp是基于ssh上实现的，所以严格来说我们是无法来关闭ssh，而只是使用sftp。
&#160; &#160;ssh默认使用的是22端口，当然这个端口是可以修改的。

#### 1.2 应用场景
&#160; &#160;局域网中有两批用户：

    一批用户：可以通过ssh登录上我们的服务器
    一批用户：可以使用sftp服务，但是我们不希望这些能够能通过ssh登录上来 

这种情况怎么处理: 我们可以将sftp用户的单独分成一个组，限制他们的操作，这个可以利用Rssh和Scponly或者实现。
但是如果提过sftp服务需要给另外一个局域网的用户使用，这样我们虽然对这些用户做了限制，我们的ssh服务还是开着的，这样他人还是可以猜我们服务器的用户名和密码，通过ssh登录上来，最好的方法是我们暴露出去的服务根本无法通过ssh登陆上来。

为了满足这两批用户的需求，我们可以再开一个ssh服务，命名为sftpd.service, 并新开一个端口号（22220），限制22220上的ssh服务只能使用sftp服务，这里利用了ssh配置文件里面的`Subsystem`,我们在里面只开sftp服务。

### 二 sftpd.service 实现

#### 2.1 拷贝sshd_config 
```sh
 cp /etc/ssh/sshd_config /etc/ssh/sftpd_config
```
#### 2.2 修改sftpd_config
```sh
Port 22220
PidFile /var/run/sftpd.pid
```
#### 2.3 拷贝 systemd 中sshd.service
```sh
cp /usr/lib/systemd/system/sshd.service  /etc/systemd/system/sftpd.service   
```

#### 2.4 修改 systemd 中sftpd.service
```sh
Description=OpenSSH sftpd instance daemon 
ExecStart=/usr/sbin/sshd -D -f /etc/ssh/sftpd_config $OPTIONS
```
#### 2.5 SELinux处理
```
semanage port -a -t ssh_port_t -p tcp 22220
```

#### 2.6 启动sftpd.service
```
systemctl enable sftpd.service
systemctl start sftpd.service
```

#### 2.7 测试
```
ssh -p 22220 user@server
```

### 三 限制服务

#### 3.1 只允许某个组下的用户使用sftpd服务
修改 /etc/ssh/sftpd_config
```sh
AllowGroup  sftponly
```

#### 3.2 关闭ssh服务
修改 /etc/ssh/sftpd_config
```
Subsystem sftp internal-sftp 
Subsystem sftp internal-sftp -l INFO -f AUTH（推荐）
```

#### 3.3 将限制的用户
修改用户的shell脚本
```
usermod -s /bin/false <username>)
```
将用户添加到sftponly组
```
usermod -a -G sftponly <username>
```

#### 3.4 测试
```
ssh sshuser@server 成功
scp -f /home/hap/test/1.txt sshuser@server:/home/hap/test 成功

ssh -p 22220 sftpuser@server 失败
scp -p 22220 -f /home/hap/test/1.txt sftpuser@server:/home/hap/test 成功
```

### 参考
[1] [How to configure multiple instances of sshd in RHEL 7](https://access.redhat.com/solutions/1166283)
[2] [Linux SSH和SFTP服务分离](https://www.cnblogs.com/zihanxing/articles/5665383.html)
