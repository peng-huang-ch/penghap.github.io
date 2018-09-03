---
title: chmod 
date: 2018-1-20
tags: [linux]
categories: linux
---

### chmod 文件权限属性设置
    chmod命令令用于改变linux系统文件或目录的访问权限。
    该命令有两种用法。一种是包含字母和操作符表达式的文字设定法；另一种是包含数字的数字设定法。

#### 数字和字母的对应关系
    读、写、运行三项权限可以用数字表示，就是r=4,w=2,x=1
    rwx = 4+2+1 = 7 可读、可写、可操作
    rw- = 4+2 = 6 可读、可写
    + :  表示增加权限、- 表示取消权限、= 表示唯一设定权限

#### 标志
| 参数 | 描述 |
| ---- | -------- |
| u | User 即文件或目录的拥有者；
| g | Group 即文件或目录的所属群组；
| o | Other 除了文件或目录拥有者或所属群组之外，其他用户皆属于这个范围；
| a | All 即全部的用户，包含拥有者，所属群组以及其他用户；
| r | 读取权限数字代号为“4”;
| w | 写入权限，数字代号为“2”；
| x | 执行或切换权限，数字代号为“1”；
| - | 不具任何权限，数字代号为“0”；
| s | 特殊功能说明：变更文件或目录的权限。

#### 文件权限
[r] 可读，[w] 可写，[x] 可执行（bash脚本等），[-]没有权限
```
[webimager@localhost ~]$ ll
总用量 105656
drwxrwxr-x.  9 webimager webimager      4096 1月  12 16:22 agent
drwxrwxr-x. 10 webimager webimager      4096 1月  12 17:21 backend
-rw-r--r--.  1 webimager webimager     41816 1月  10 14:36 curlftpfs-0.9.2-14.el7.x86_64.rpm
drwxr-xr-x.  2 webimager webimager         6 1月  16 16:26 Desktop
drwxr-xr-x.  2 webimager webimager         6 1月   4 19:24 Documents
drwxr-xr-x.  2 webimager webimager        56 1月  16 15:47 Downloads
drwxr-xr-x.  2 webimager webimager         6 1月   4 19:24 Music
drwxr-xr-x.  2 webimager webimager      4096 1月  20 15:30 Pictures
drwxr-xr-x.  2 webimager webimager         6 1月   4 19:24 Public
drwxrwxr-x.  3 webimager webimager        37 1月  16 17:19 taskData
drwxr-xr-x.  2 webimager webimager         6 1月   4 19:24 Templates
drwxr-xr-x.  2 webimager webimager         6 1月   4 19:24 Videos
-rw-r--r--.  1 webimager webimager 108133225 1月  12 16:20 WebImager_Online_2.2.0.2041.zip
```
一栏会有十个字符
- 第一个字符 文件的类型

| 符号 | 描述 |
| ---- | -------- |
| d | 目录（文件夹） |
| - | 文件 |
| l | 链接 |
| b | 接口设备 |
| c | 串行端口设备（键盘、鼠标等） |

- 第二到四个字符 文件拥有者的权限
- 第五到七个字符 同群组的权限
- 第八到十个字符 其他非本群组的权限

#### 例子
```
chmod u+x,g+w task.log　　//为文件task.log设置自己可以执行，组员可以写入的权限
chmod u=rwx,g=rw,o=r task.log //为文件task.log设置自己可读可写可操作，组员可读可写，其他人可读
chmod 764 task.log //与上面等价
chmod a+x f01　　//对文件task.log的u,g,o都设置可执行属性
```
