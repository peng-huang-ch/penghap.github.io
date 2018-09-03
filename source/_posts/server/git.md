---
title: git基本使用
date: 2018-03-13
tags: [git, linux]
categories: linux
---
### 一、 [介绍](https://git-scm.com/)

Git是一个免费的开源分布式版本控制系统。

### 二、基本命令

#### 2.1、初始化代码仓库
git init 
```sh
 /wwwroot/gittest  git init
Initialized empty Git repository in /wwwroot/gittest/.git/

/wwwroot/gittest   master  ls -al
total 0
drwxr-xr-x   3 xxxxx  wheel   96  5 13 19:18 .
drwxr-xr-x  24 xxxxx  wheel  768  5 13 19:18 ..
drwxr-xr-x  10 xxxxx  wheel  320  5 13 21:33 .git
```

#### 2.2、查看当前git仓的状态
```sh
git status
```
![image](/images/git/status.png)

#### 2.3、跟踪新文件
```sh
git add
```
![image](/images/git/add.png)

#### 2.4、提交更新
```sh
git commit -m <message>
```
![image](/images/git/add_commit.png)

```sh
git commit
```
![image](/images/git/commit_mult.png)

#### 2.5、暂存

##### 2.5.1、添加暂存
```sh
git stash
```
![image](/images/git/stash.png)

##### 2.5.2、查看已有的暂存
```sh
git stash list
stash@{0}: WIP on master: 2ba5560 added test file
stash@{1}: WIP on master: c49d078 another the index file
```

##### 2.5.3、恢复暂存
```sh
git stash pop [–index] [stash_id] -- 应用及删除
git stash pop stash@{1}
```
![image](/images/git/stash_apply_drop.png)

```sh
git stash apply [–index] [stash_id]  -- 应用
git stash apply stash@{0}

git stash drop [stash_id]  -- 删除
git stash apply stash@{0}
```
![image](/images/git/stash_pop.png)

##### 2.5.4、删除所有暂存
```sh
git stash clear
```

#### 2.6、查看修改
```sh
git diff 
git difftool
```
![image](/images/git/diff.png)

MAC 下 [BeyondCompare](https://www.scootersoftware.com/support.php?zz=kb_vcs)配置
```
vim ~/.gitconfig
```
```
[diff]
  tool = bcomp
[difftool]
  prompt = false
[difftool "bcomp"]
  trustExitCode = true
  cmd = "/usr/local/bin/bcomp" \"$LOCAL\" \"$REMOTE\"
[merge]
  tool = bcomp
[mergetool]
  prompt = false
[mergetool "bcomp"]
  trustExitCode = true
  cmd = "/usr/local/bin/bcomp" \"$LOCAL\" \"$REMOTE\" \"$BASE\" \"$MERGED\"
[user]
  name = ***
  email = ***
```


#### 2.7、分支

##### 2.7.1、添加分支
```sh
git brach <new brach>
```

##### 2.7.2、查看所有分支
```sh
git brach
```

##### 2.7.3、切换分支
```sh
git checkout dev
```

##### 2.7.4、删除分支
```sh
git brach -d dev
git brach -D dev 强制删除
```
![image](/images/git/branch.png)

#### 2.8、合并分支
```sh
git merge <branch>
```
![image](/images/git/merge.png)

#### 2.9、标签
##### 2.9.1、添加tag
```sh
git tag -a <tag> -m <message>
```
```sh
git tag -a 'v0.1' -m 'v0.1'
```

##### 2.9.2、显示tag
```sh
git tag
v0.1
v0.2
```

##### 2.9.3、查看标签信息
```sh
git show <tag>
```
![image](/images/git/tag.png)

#### 2.10、移除文件
```sh
git rm -- <file>
```

#### 2.11 从现有仓库克隆
```sh
git clone [url]
```

#### 2.12 拉取代码
```sh
git pull [options] [<repository> [<refspec>…]]
git pull <远程主机名> <远程分支名>:<本地分支名>

git pull --rebase origin dev
```
#### 2.13 远程仓库

#### 2.13.1、查看远程仓库
```sh
git remote show master
```
#### 2.13.2、推送到远程仓库
```sh
git push [remote-name] [branch-name]
git push origin dev
```

#### 2.13.3、移除与重命名
```sh
git remote rename <branch> <new branch>
git remote rename dev dev1
```

### 参考文件
[1] [Git-基础](https://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80)
[2] [Git常用命令](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
[3] [Git教程](https://www.yiibai.com/git/)
[4] [Git 分支 - 变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)