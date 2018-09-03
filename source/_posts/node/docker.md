---
title: docker
date: 2018-03-03
tags: [服务器, node]
categories: node
---
### 一、 介绍

### 二、安装docker
https://store.docker.com/editions/community/docker-ce-desktop-mac

### 三、启动virtualbox
```sh
docker-machine create --driver virtualbox default
```
如出现如下错误：
```sh
docker-machine create default
Running pre-create checks...
(default) Default Boot2Docker ISO is out-of-date, downloading the latest release...
(default) Latest release for github.com/boot2docker/boot2docker is v18.03.1-ce
(default) Downloading /Users/huangpeng/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.03.1-ce/boot2docker.iso...
Error with pre-create check: "Get https://github-production-release-asset-2e65be.s3.amazonaws.com/14930729/be3dfbf8-4936-11e8-88a6-1d0cd712540c?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20180508%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180508T142908Z&X-Amz-Expires=300&X-Amz-Signature=3d383ae9b91f458417e8032ef413e42fce3502b21f1bef4fd212089cb2f97e86&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dboot2docker.iso&response-content-type=application%2Foctet-stream: dial tcp 52.216.65.32:443: getsockopt: operation timed out"
```
[解决方法]
(https://github.com/docker/machine/issues/3229)
可手动下载ISO文件，然后移动到
```sh
/Users/{user}/.docker/machine/cache/
```

再次运行
```
docker-machine create --driver virtualbox default
```

### 四、docker启动node

#### 4.1、安装node和mongodb
查询mongo镜像
```
~$ docker search mongo
NAME                                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mongo                               MongoDB document databases provide high av...   4428                [OK]
mongo-express                       Web-based MongoDB admin interface, written...   247                 [OK]
tutum/mongodb                       MongoDB Docker image – listens in port 270...   224                                     [OK]
mvertes/alpine-mongo                light MongoDB container                         74                                      [OK]
mongoclient/mongoclient             Official docker image for Mongoclient, fea...   50                                      [OK]
```

安装mongo
```sh
~$ docker pull mongo
Using default tag: latest
latest: Pulling from library/mongo
4d0d76e05f3c: Pull complete
2da2ecd7fdbd: Pull complete
c3a86da34d0f: Pull complete
e2b1f447e420: Pull complete
c9e820834b36: Pull complete
ffa34fa64bf4: Pull complete
63127ea58ee0: Pull complete
ccb46836c598: Pull complete
7b0abf374ec4: Pull complete
0e8b13c8fd38: Pull complete
Digest: sha256:c6d2b2f8c054210db26b492bab81ffab171ee54eb58925fa98fabb4faca3a9cb
Status: Downloaded newer image for mongo:latest
```

查询node镜像
```sh
~$ docker search node
NAME                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
node                                   Node.js is a JavaScript-based platform for...   5513                [OK]
mhart/alpine-node                      Minimal Node.js built on Alpine Linux           355
mongo-express                          Web-based MongoDB admin interface, written...   247                 [OK]
nodered/node-red-docker                Node-RED Docker images.                         140                                     [OK]
iojs                                   io.js is an npm compatible platform origin...   125                 [OK]
prom/node-exporter
```
安装node
```sh
~$ docker pull node
Using default tag: latest
latest: Pulling from library/node
3d77ce4481b1: Pull complete
534514c83d69: Pull complete
d562b1c3ac3f: Pull complete
4b85e68dc01d: Pull complete
f6a66c5de9db: Pull complete
7a4e7d9a081d: Pull complete
d5019a4c5f9e: Pull complete
dbeca1767f60: Pull complete
Digest: sha256:4013aa6c297808defd01234fce4a42e1ca0518a5bd0260752a86a46542b38206
Status: Downloaded newer image for node:latest
```
#### 4.2、查看已安装镜像
```sh
~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
node                latest              1c1272350058        3 days ago          675MB
mongo               latest              14c497d5c758        7 days ago          366MB
```

#### 4.3、创建Dockerfile、.dockerignore
Dockerfile
```sh
# 指定node镜像
FROM node
# 指定制作人（镜像创建者）
MAINTAINER hap

# 将根目录下的文件都copy到container（运行此镜像的容器）文件系统的app文件夹下
ADD . /myapp/
# cd到app文件夹下
WORKDIR /myapp

# 安装项目依赖包 --productions https://docs.npmjs.com/cli/install
RUN npm install --productions --registry=https://registry.npm.taobao.org

RUN npm install pm2 -g --registry=https://registry.npm.taobao.org

# 配置环境变量
ENV HOST 0.0.0.0
ENV PORT 3000

# 容器对外暴露的端口号
EXPOSE 3000

# 容器启动时执行的命令，类似npm run start, node app.js  --no-daemon
CMD pm2 start app.js --no-daemon

```
.dockerignore
```sh
.DS_Store
*.log
*.tar
*.md
*.zip
/package-lock.json
```

#### 4.4、创建express
expresss
```js
'use strict';

const express = require('express');
const db = require('./db');

const app = express();

app.get('/', (req, res) => {
  res.send('hello node');
});

app.listen(3000);
console.log('监听3000端口号')
```

db
```js
'use strict';

const mongoose = require('mongoose');

let mongodbUri = 'mongodb://mongo:27017/myapp'
let options = {
  autoReconnect: true
};

mongoose.connect(mongodbUri, options);
mongoose.Promise = global.Promise;

const db = mongoose.connection;

db.once('open', () => {
  console.log('连接数据库成功');
})

db.on('error', function(error) {
  console.error('connection error: ' + error);
  mongoose.disconnect();
});

db.on('close', function() {
  console.log('数据库断开，重新连接数据库');
  mongoose.connect(mongodbUri, options);
});

module.exports = db;
```

#### 4.5、构建镜像

cd workpace 进入工作区

构建镜像
`docker build -t myapp .` 
```sh
dokcer$ docker build -t myapp .
Sending build context to Docker daemon  48.42MB
Step 1/10 : FROM node
 ---> 1c1272350058
Step 2/10 : MAINTAINER hap
 ---> Running in 30e3f239873c
Removing intermediate container 30e3f239873c
 ---> 91da1a62b412
Step 3/10 : ADD . /myapp/
 ---> 761686544de8
Step 4/10 : WORKDIR /myapp
Removing intermediate container 3c6da81e12b0
 ---> d2eb5687c486
Step 5/10 : RUN npm install --productions
 ---> Running in c924d3f9eb96
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN docker@1.0.0 No description
npm WARN docker@1.0.0 No repository field.

up to date in 4.934s
Removing intermediate container c924d3f9eb96
.
.
.
+ pm2@2.10.3
added 222 packages in 30.691s
Removing intermediate container 77cb631c1385
 ---> 0d91c7643b3a
Step 7/10 : ENV HOST 0.0.0.0
 ---> Running in 313c1ee1adc1
Removing intermediate container 313c1ee1adc1
 ---> 09e643f171df
Step 8/10 : ENV PORT 3000
 ---> Running in 9ea98a37cd09
Removing intermediate container 9ea98a37cd09
 ---> aeaa809ff9d8
```

查看镜像
```sh
dokcer$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
myapp               latest              b48934a3193c        2 minutes ago       737MB
node                latest              1c1272350058        3 days ago          675MB
mongo               latest              14c497d5c758        7 days ago          366MB
nginx               latest              ae513a47849c        8 days ago          109MB
```

#### 4.6、启动mongodb
```sh
sudo mkdir -p /data/db

启动mongo
dokcer$ docker run -v /data/db:/data/db -p 28017:27017 --name mongo -d mongo --smallfiles

查看已启动容器
dokcer$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
24f2d5c03825        mongo               "docker-entrypoint..."   10 seconds ago      Up 9 seconds        0.0.0.0:28017->27017/tcp   mongo

查看所有容器
dokcer$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
24f2d5c03825        mongo               "docker-entrypoint..."   44 seconds ago      Up 44 seconds       0.0.0.0:28017->27017/tcp   mongo

进入容器（或docker exec -it [CONTAINER ID] bash）
dokcer$ docker exec -it mongo bash
root@24f2d5c03825:/# mongo
MongoDB shell version v3.6.4
connecting to: mongodb://127.0.0.1:27017
```

##### 4.6.1、数据库备份
  
1、进入数据库容器

语法

```sh
docker exec -it <container id> /bin/bash
docker exec -it <container id> bash
```
```
docker exec -it 2e10fd885cc0 bash
~ docker exec -it 2e10fd885cc0 bash
root@2e10fd885cc0:/#
```

2、备份数据库(mongodump)

语法

```
mongodump -h <dbhost> --port <port> -d <db> -o <dbdirectory>
```

```sh
root@2e10fd885cc0:/# mongodump -h 127.0.0.1 --port 27017 -d test -o /dump/mongodb
2018-05-13T07:11:24.533+0000	writing test.test to
2018-05-13T07:11:24.534+0000	done dumping test.test (2 documents)
```

| 参数 | 作用 |
| ---- | ---- |
| -h   | host |
| --port | 端口号，默认 27017 |
| -d | 指定数据库 |
| -o | 指定备份到哪个目录 |
| -u | 用户名 |
| -p | 密码 |

3、压缩

```sh
tar -zcvf test.tar.gz /dump/mongodb/test

root@2e10fd885cc0:/# tar -zcvf /dump/mongodb/test.tar.gz /dump/mongodb/test
tar: Removing leading `/' from member names
/dump/mongodb/test/
/dump/mongodb/test/test.bson
/dump/mongodb/test/test.metadata.json
```

4、拷贝备份文件到外部
docker cp :用于容器与主机之间的数据拷贝。

语法
```sh
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

```sh
docker cp 2e10fd885cc0:/dump/mongodb/test.tar.gz /wwwroot
```

##### 4.6.2、数据库恢复

1、拷贝外部文件到容器
```
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

docker cp /wwwroot/test.tar.gz 6608bd0b581c:/dump
```
2、解压文件

```sh
tar -zxvf
```

```sh 
root@6608bd0b581c:/dump# tar -zxvf /dump/test.tar.gz
dump/mongodb/test/
dump/mongodb/test/test.bson
dump/mongodb/test/test.metadata.json
```

3、数据库恢复

```sh
mongorestore -h <hostname><:port> -d dbname <path>
```

```sh
root@6608bd0b581c:/dump# mongorestore -d test --dir mongodb/test/
2018-05-13T08:29:19.155+0000	the --db and --collection args should only be used when restoring from a BSON file. Other uses are deprecated and will not exist in the future; use --nsInclude instead
2018-05-13T08:29:19.155+0000	building a list of collections to restore from mongodb/test dir
2018-05-13T08:29:19.156+0000	reading metadata for test.test from mongodb/test/test.metadata.json
2018-05-13T08:29:19.159+0000	restoring test.test from mongodb/test/test.bson
2018-05-13T08:29:19.161+0000	no indexes to restore
2018-05-13T08:29:19.161+0000	finished restoring test.test (2 documents)
2018-05-13T08:29:19.161+0000	done
```

4、查看恢复结果

![image](/images/docker/mongorestore.png)

mongorestore.png

#### 4.7、运行myapp进行

##### 4.7.1、--link 已不推荐了(https://docs.docker.com/network/links/)
```sh
docker run -d -p 3000:3000 --link mongo:mongo myapp
```
##### 4.7.2、[使用network运行](https://docs.docker.com/network/bridge/#connect-a-container-to-a-user-defined-bridge)
创建网络
```
docker network create my-net
```

查看
```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
ab5ea87cc7c0        bridge              bridge              local
102eb50cfda7        host                host                local
4f8229f2170c        my-net              bridge              local
033fb19b6535        none                null                local
```

运行是关联网络
```
docker run -v /data/db:/data/db --net=my-net -p 28017:27017 --name mongo -d mongo --smallfiles
```

关联已经运行程序
```
docker network connect my-net mongo
```

```
docker run -d -p 3000:3000 --net=my-net myapp
```

##### 4.7.3、使用docker-compose待完善)
1. [mongo](https://hub.docker.com/_/mongo/)
2. [docker-volumes-and-networks-compose](https://www.linux.com/learn/docker-volumes-and-networks-compose)
3. [探究Docker Stack和可对接网络](http://www.dockerinfo.net/4245.html)
4. [dockerize-nodejs-service-with-mongodb-docker-compose/](https://ciphertrick.com/2017/10/23/dockerize-nodejs-service-with-mongodb-docker-compose/)

#### 4.8、进入容器里面
```sh
docker exec -it 5af9a8144595 bash
```
![image](/images/docker/mongo.png)

#### 4.9、查看当前ip
```sh
docker-machine ip default
192.168.99.100
```
```sh
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container id>

~$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' 24f2d5c03825
172.17.0.2
```
```sh
docker inspect <container id> | grep "IPAddress"

~$ docker inspect 24f2d5c03825 | grep "IPAddress"
  "SecondaryIPAddresses": null,
  "IPAddress": "172.17.0.2",
  "IPAddress": "172.17.0.2",
```

查看所有容器的IP
```sh
docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```
![image](/images/docker/curl.png)

#### 4.10、 本地访问测试

![image](/images/docker/exec.png)

```sh
curl 127.0.0.1:3000/
root@5af9a8144595:/myapp# curl 127.0.0.1:3000/
hello node
```
![image](/images/docker/app.png)

#### 4.11、 同步主机时区

```sh
docker run -v /etc/localtime:/etc/localtime <IMAGE:TAG>
```
进入已运行的容器
```
docker exec -it <CONTAINER NAME> bash

~$ docker exec -it 5af9a8144595  bash

root@5af9a8144595:/myapp# date
Sun May 13 14:33:26 CST 2018

root@5af9a8144595:/myapp# echo "Asia/Shanghai" > /etc/timezone
root@5af9a8144595:/myapp# dpkg-reconfigure -f noninteractive tzdata

Current default time zone: 'Asia/Shanghai'
Local time is now:      Sun May 13 14:33:20 CST 2018.
Universal Time is now:  Sun May 13 06:33:20 UTC 2018.

```

### 五、常用命令
```sh
attach    Attach to a running container                 # 当前 shell 下 attach 连接指定运行镜像
build     Build an image from a Dockerfile              # 通过 Dockerfile 定制镜像
commit    Create a new image from a container changes   # 提交当前容器为新的镜像
cp        Copy files/folders from the containers filesystem to the host path   # 从容器中拷贝指定文件或者目录到宿主机中
create    Create a new container                        # 创建一个新的容器，同 run，但不启动容器
diff      Inspect changes on a container filesystem     # 查看 docker 容器变化
events    Get real time events from the server          # 从 docker 服务获取容器实时事件
exec      Run a command in an existing container        # 在已存在的容器上运行命令
export    Stream the contents of a container as a tar archive   
# 导出容器的内容流作为一个 tar 归档文件[对应 import ]
history   Show the history of an image                  # 展示一个镜像形成历史
images    List images                                   # 列出系统当前镜像
import    Create a new filesystem image from the contents of a tarball    # 从tar包中的内容创建一个新的文件系统映像[对应 export]
info      Display system-wide information               # 显示系统相关信息
inspect   Return low-level information on a container   # 查看容器详细信息
kill      Kill a running container                      # kill 指定 docker 容器
load      Load an image from a tar archive              # 从一个 tar 包中加载一个镜像[对应 save]
login     Register or Login to the docker registry server    
# 注册或者登陆一个 docker 源服务器
logout    Log out from a Docker registry server        # 从当前 Docker registry 退出
logs      Fetch the logs of a container                 # 输出当前容器日志信息
port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT    # 查看映射端口对应的容器内部源端口
pause     Pause all processes within a container        # 暂停容器
ps        List containers                               # 列出容器列表
pull      Pull an image or a repository from the docker registry server   # 从docker镜像源服务器拉取指定镜像或者库镜像
push      Push an image or a repository to the docker registry server    # 推送指定镜像或者库镜像至docker源服务器
restart   Restart a running container                   # 重启运行的容器
rm        Remove one or more containers                 # 移除一个或者多个容器
rmi       Remove one or more images                     # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
run       Run a command in a new container              # 创建一个新的容器并运行一个命令
save      Save an image to a tar archive                # 保存一个镜像为一个 tar 包[对应 load]
search    Search for an image on the Docker Hub         # 在 docker hub 中搜索镜像
start     Start a stopped containers                    # 启动容器
stop      Stop a running containers                     # 停止容器
tag       Tag an image into a repository                # 给源中镜像打标签
top       Lookup the running processes of a container   # 查看容器中运行的进程信息
unpause   Unpause a paused container                    # 取消暂停容器
version   Show the docker version information           # 查看 docker 版本号
wait      Block until a container stops, then print its exit code   
# 截取容器停止时的退出状态值
```

### 参考文件
1. [how-to-mongodb-nodejs-docker](http://www.ifdattic.com/how-to-mongodb-nodejs-docker/)
2. [Docker 镜像加速器](https://yq.aliyun.com/articles/29941?spm=5176.10695662.1996646101.searchclickresult.331f64a3HNpdWe)
3. [Docker — 从入门到实践](https://github.com/yeasy/docker_practice)
4. [connect-a-container-to-a-user-defined-bridge](https://docs.docker.com/network/bridge/#connect-a-container-to-a-user-defined-bridge)
5. [https://docs.docker.com/network/bridge/](https://docs.docker.com/network/bridge/)