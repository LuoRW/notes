教程：https://www.bilibili.com/video/BV1CJ411T7BK/

## 初识 Docker

### docker概念

- Docker 是一个开源的应用容器引擎
- 诞生于 2013 年初，基于 Go 语言实现， dotCloud 公司出品（后改名为Docker Inc）
- Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux机器上。
- 容器是完全使用沙箱机制，相互隔离
- 容器性能开销极低。
- Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版）

**软件跨环境迁移的问题，“水土”不服**



### 安装docker

```shell
# 1、yum 包更新到最新 
yum update
# 2、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的 
yum install -y yum-utils device-mapper-persistent-data lvm2
# 3、 设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 4、 安装docker，出现输入的界面都按 y 
yum install -y docker-ce
# 5、 查看docker版本，验证是否验证成功
docker -v
```



### 配置阿里云镜像加速

默认情况下，将来从docker hub（https://hub.docker.com/）上下载docker镜像，太慢。一般都会配置镜像加速器

![image-20211219214820467](https://gitee.com/L-rw/img/raw/master/image-20211219214820467.png)



### docker架构

![image-20211219214909168](https://gitee.com/L-rw/img/raw/master/image-20211219214909168.png)

- **镜像（Image）**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
- **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和对象一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- **仓库（Repository）**：仓库可看成一个代码控制中心，用来保存镜像。

## Docker 命令

### 进程相关命令

- 启动docker服务 `systemctl start docker `
- 停止docker服务 `systemctl stop docker `
- 重启docker服务 `systemctl restart docker`
- 查看docker服务状态 `systemctl status docker `
- 开机启动docker服务 `systemctl enable docker`



### 镜像相关命令

- 查看镜像: 查看本地所有的镜像

  ```shell
  docker images
  docker images –q # 查看所用镜像的id
  ```

- 搜索镜像:从网络中查找需要的镜像

  ```shell
  docker search 镜像名称
  ```

- 拉取镜像:从Docker仓库下载镜像到本地，镜像名称格式为 名称:版本号，如果版本号不指定则是最新的版本。如果不知道镜像版本，可以去docker hub 搜索对应镜像查看。

  ```shell
  docker pull 镜像名称
  ```

- 删除镜像: 删除本地镜像

  ```shell
  docker rmi 镜像id # 删除指定本地镜像
  docker rmi `docker images -q`  # 删除所有本地镜像
  ```



### 容器相关命令

- 查看容器

  ```shell
  docker ps # 查看正在运行的容器
  docker ps –a # 查看所有容器
  ```

- 创建容器

  ```shell
  docker run 参数
  ```

  参数说明：

  - -i：保持容器运行。通常与 -t 同时使用。加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。
  - -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用。
  - -d：以守护（后台）模式运行容器。创建一个容器在后台运行，需要使用docker exec 进入容器。退出后，容器不会关闭。
  - -it 创建的容器一般称为交互式容器，-id 创建的容器一般称为守护式容器
  - --name：为创建的容器命名。

- 进入容器

  ```shell
  docker exec 参数 # 退出容器，容器不会关闭
  ```

- 启动容器

  ```shell
  docker start 容器名称
  ```

- 停止容器

  ```shell
  docker stop 容器名称
  ```

- 删除容器

  ```shell
  docker rm 容器名称
  ```

- 查看容器信息

  ```shell
  docker inspect 容器名称
  ```



## Docker 容器数据卷

### 数据卷概念及作用

![image-20211219220716486](https://gitee.com/L-rw/img/raw/master/image-20211219220716486.png)

**概念：**

- 数据卷是宿主机中的一个目录或文件



**作用**

- 容器数据持久化
- 外部机器和容器间接通信
- 容器之间数据交换



### 配置数据卷

- 创建启动容器时，使用 –v 参数 设置数据卷

  ```shell
  docker run ... –v 宿主机目录(文件):容器内目录(文件) ... 
  ```

- 注意事项： 
  1. 目录必须是绝对路径
  2. 如果目录不存在，会自动创建
  3. 可以挂载多个数据卷



### 配置数据卷容器

多容器进行数据交换

1. 多个容器挂载同一个数据卷

2. 数据卷容器

![image-20211219221221027](https://gitee.com/L-rw/img/raw/master/image-20211219221221027.png)



1. 创建启动c3数据卷容器，使用 –v 参数 设置数据卷

   ```shell
   docker run –it --name=c3 –v /volume centos:7 /bin/bash 
   ```

2. 创建启动 c1 c2 容器，使用 –-volumes-from 参数 设置数据卷

   ```shell
   docker run –it --name=c1 --volumes-from c3 centos:7 /bin/bash
   
   docker run –it --name=c2 --volumes-from c3 centos:7 /bin/bash  
   ```



## Docker 应用部署

### 部署MySQL

- 容器内的网络服务和外部机器不能直接通信
- 外部机器和宿主机可以直接通信
- 宿主机和容器可以直接通信
- 当容器中的网络服务需要被外部机器访问时，可以将容器中提供服务的端口映射到宿主机的端口上。外部机器访问宿主机的该端口，从而间接访问容器的服务。
- 这种操作称为：**端口映射**

![image-20211219230250280](https://gitee.com/L-rw/img/raw/master/image-20211219230250280.png)





1. 搜索mysql镜像

   ```shell
   docker search mysql
   ```

2. 拉取mysql镜像

   ```shell
   docker pull mysql:5.7
   ```

3. 创建容器，设置端口映射、目录映射

   ```shell
   # 在/root目录下创建mysql目录用于存储mysql数据信息
   mkdir ~/mysql
   cd ~/mysql
   ```

   ```shell
   docker run -id \
   -p 3307:3306 \
   --name=c_mysql \
   -v $PWD/conf:/etc/mysql/conf.d \
   -v $PWD/logs:/logs \
   -v $PWD/data:/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=123456 \
   mysql:5.7
   ```

- 参数说明：
  - **-p 3307:3306**：将容器的 3306 端口映射到宿主机的 3307 端口。
  - **-v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。配置目录
  - **-v $PWD/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。日志目录
  - **-v $PWD/data:/var/lib/mysql** ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。数据目录
  - **-e MYSQL_ROOT_PASSWORD=123456：**初始化 root 用户的密码。

4. 进入容器，操作mysql

   ```shell
   docker exec –it c_mysql /bin/bash
   ```

5. 使用外部机器连接容器中的mysql

![1573636765632](https://gitee.com/L-rw/img/raw/master/1573636765632.png)




### 部署Tomcat

1. 搜索tomcat镜像

   ```shell
   docker search tomcat
   ```

2. 拉取tomcat镜像

   ```shell
   docker pull tomcat
   ```

3. 创建容器，设置端口映射、目录映射

   ```shell
   # 在/root目录下创建tomcat目录用于存储tomcat数据信息
   mkdir ~/tomcat
   cd ~/tomcat
   ```

   ```shell
   docker run -id --name=c_tomcat \
   -p 8080:8080 \
   -v $PWD:/usr/local/tomcat/webapps \
   tomcat 
   ```

- 参数说明：

  - **-p 8080:8080：**将容器的8080端口映射到主机的8080端口

    **-v $PWD:/usr/local/tomcat/webapps：**将主机中当前目录挂载到容器的webapps



4. 使用外部机器访问tomcat

![1573649804623](https://gitee.com/L-rw/img/raw/master/1573649804623.png)




### 部署Nginx

1. 搜索nginx镜像

   ```shell
   docker search nginx
   ```

2. 拉取nginx镜像

   ```shell
   docker pull nginx
   ```

3. 创建容器，设置端口映射、目录映射


   ```shell
   # 在/root目录下创建nginx目录用于存储nginx数据信息
   mkdir ~/nginx
   cd ~/nginx
   mkdir conf
   cd conf
   # 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容
   vim nginx.conf
   ```

   ```shell
   user  nginx;
   worker_processes  1;
  
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;
  
  
   events {
       worker_connections  1024;
   }

  
   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;
  
       log_format  main  '$remote_addr - $remote_user [$time_local]   "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
  
       access_log  /var/log/nginx/access.log  main;
  
       sendfile        on;
       #tcp_nopush     on;
  
       keepalive_timeout  65;
  
       #gzip  on;
  
       include /etc/nginx/conf.d/*.conf;
   }
  
  
   ```


   ```shell
   docker run -id --name=c_nginx \
   -p 80:80 \
   -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
   -v $PWD/logs:/var/log/nginx \
   -v $PWD/html:/usr/share/nginx/html \
   nginx
   ```

- 参数说明：
  - **-p 80:80**：将容器的 80端口映射到宿主机的 80 端口。
  - **-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf**：将主机当前目录下的 /conf/nginx.conf 挂载到容器的 :/etc/nginx/nginx.conf。配置目录
  - **-v $PWD/logs:/var/log/nginx**：将主机当前目录下的 logs 目录挂载到容器的/var/log/nginx。日志目录

4. 使用外部机器访问nginx

![1573652396669](https://gitee.com/L-rw/img/raw/master/1573652396669.png)



### 部署Redis

1. 搜索redis镜像

   ```shell
   docker search redis
   ```

2. 拉取redis镜像

   ```shell
   docker pull redis:5.0
   ```

3. 创建容器，设置端口映射

   ```shell
    docker run -id --name=c_redis -p 6379:6379 redis:5.0
   ```

4. 使用外部机器连接redis

   ```shell
   ./redis-cli.exe -h 192.168.149.135 -p 6379
   ```



## Dockerfile

### Docker 镜像原理

- Docker镜像是由特殊的文件系统叠加而成
- 最底端是 bootfs，并使用宿主机的bootfs 
- 第二层是 root文件系统rootfs,称为base image
- 然后再往上可以叠加其他的镜像文件
- 统一文件系统（Union File System）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。
- 一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像成为基础镜像。
- 当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器

![image-20211219224140065](https://gitee.com/L-rw/img/raw/master/image-20211219224140065.png)



1. Docker 镜像本质是什么？

   是一个分层文件系统

2. Docker 中一个centos镜像为什么只有200MB，而一个centos操作系统的iso文件要几个个G？

   Centos的iso镜像文件包含bootfs和rootfs，而docker的centos镜像复用操作系统的bootfs，只有rootfs和其他镜像层

3. Docker 中一个tomcat镜像为什么有500MB，而一个tomcat安装包只有70多MB？

   由于docker中镜像是分层的，tomcat虽然只有70多MB，但他需要依赖于父镜像和基础镜像，所有整个对外暴露的tomcat镜像大小500多MB



### 镜像制作

![image-20211219224504123](https://gitee.com/L-rw/img/raw/master/image-20211219224504123.png)

1. 容器转为镜像

   ```shell
   docker commit 容器id 镜像名称:版本号
   docker save -o 压缩文件名称 镜像名称:版本号
   docker load –i 压缩文件名称
   ```

2. dockerfile



### Dockerfile概念及作用

- Dockerfile 是一个文本文件
- 包含了一条条的指令
- 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
- 对于开发人员：可以为开发团队提供一个完全一致的开发环境
- 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了
- 对于运维人员：在部署时，可以实现应用的无缝移植



Dochub网址：[https://hub.docker.com](https://hub.docker.com/) 



### Dockerfile关键字

| 关键字      | 作用                     | 备注                                                         |
| ----------- | ------------------------ | ------------------------------------------------------------ |
| FROM        | 指定父镜像               | 指定dockerfile基于那个image构建                              |
| MAINTAINER  | 作者信息                 | 用来标明这个dockerfile谁写的                                 |
| LABEL       | 标签                     | 用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看 |
| RUN         | 执行命令                 | 执行一段命令 默认是/bin/sh 格式: RUN command 或者 RUN ["command" , "param1","param2"] |
| CMD         | 容器启动命令             | 提供启动容器时候的默认命令 和ENTRYPOINT配合使用.格式 CMD command param1 param2 或者 CMD ["command" , "param1","param2"] |
| ENTRYPOINT  | 入口                     | 一般在制作一些执行就关闭的容器中会使用                       |
| COPY        | 复制文件                 | build的时候复制文件到image中                                 |
| ADD         | 添加文件                 | build的时候添加文件到image中 不仅仅局限于当前build上下文 可以来源于远程服务 |
| ENV         | 环境变量                 | 指定build时候的环境变量 可以在启动的容器的时候 通过-e覆盖 格式ENV name=value |
| ARG         | 构建参数                 | 构建参数 只在构建的时候使用的参数 如果有ENV 那么ENV的相同名字的值始终覆盖arg的参数 |
| VOLUME      | 定义外部可以挂载的数据卷 | 指定build的image那些目录可以启动的时候挂载到文件系统中 启动容器的时候使用 -v 绑定 格式 VOLUME ["目录"] |
| EXPOSE      | 暴露端口                 | 定义容器运行的时候监听的端口 启动容器的使用-p来绑定暴露端口 格式: EXPOSE 8080 或者 EXPOSE 8080/udp |
| WORKDIR     | 工作目录                 | 指定容器内部的工作目录 如果没有创建则自动创建 如果指定/ 使用的是绝对地址 如果不是/开头那么是在上一条workdir的路径的相对路径 |
| USER        | 指定执行用户             | 指定build或者启动的时候 用户 在RUN CMD ENTRYPONT执行的时候的用户 |
| HEALTHCHECK | 健康检查                 | 指定监测当前容器的健康监测的命令 基本上没用 因为很多时候 应用本身有健康监测机制 |
| ONBUILD     | 触发器                   | 当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后 会执行 ONBUILD的命令 但是不影响当前镜像 用处也不怎么大 |
| STOPSIGNAL  | 发送信号量到宿主机       | 该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出。       |
| SHELL       | 指定执行脚本的shell      | 指定RUN CMD ENTRYPOINT 执行命令的时候 使用的shell            |



### 案例

1. 需求

   自定义centos7镜像。要求：

   1. 默认登录路径为 /usr

   2. 可以使用vim

2. 实现步骤

   ①定义父镜像：`FROM centos:7`

   ②定义作者信息：`MAINTAINER  itheima <itheima@itcast.cn>`

   ③执行安装vim命令： `RUN yum install -y vim`

   ④定义默认的工作目录：`WORKDIR /usr`

   ⑤定义容器启动执行的命令：`CMD /bin/bash`

   ⑥通过dockerfile构建镜像：docker bulid –f dockerfile文件路径 –t 镜像名称:版本

```bash
FROM centos:7
MAINTAINER itheima <itheima@itcast.cn>
RUN yum install -y vim
WORKDIR /usr
CMD /bin/bash
```



1. 需求

   定义dockerfile，发布springboot项目

2. 实现步骤

   ①定义父镜像：`FROM java:8`

   ②定义作者信息：`MAINTAINER  itheima <itheima@itcast.cn>`

   ③将jar包添加到容器： `ADD springboot.jar app.jar`

   ④定义容器启动执行的命令：`CMD java–jar app.jar`

   ⑤通过dockerfile构建镜像：docker bulid –f dockerfile文件路径 –t 镜像名称:版本



## Docker 服务编排

### 服务编排概念

按照一定的业务规则批量管理容器



### Docker Compose 概述

Docker Compose是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止。使用步骤：

1. 利用 Dockerfile 定义运行环境镜像

2. 使用 docker-compose.yml 定义组成应用的各服务

3. 运行 docker-compose up 启动应用



### Docker Compose 安装使用

**安装Docker Compose**

   ```shell
   # Compose目前已经完全支持Linux、Mac OS和Windows，在我们安装Compose之前，需要 先安装Docker。下面我 们以编译好的二进制包方式安装在Linux系统中。 
   curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose- `uname -s`-`uname -m` -o /usr/local/bin/docker-compose
   # 设置文件可执行权限 
   chmod +x /usr/local/bin/docker-compose
   # 查看版本信息 
   docker-compose -version
   ```



**卸载Docker Compose**

   ```shell
   # 二进制包方式安装的，删除二进制文件即可
   rm /usr/local/bin/docker-compose
   ```



### 案例

**使用docker compose编排nginx+springboot项目**

1. 创建docker-compose目录

   ```shell
   mkdir ~/docker-compose
   cd ~/docker-compose
   ```

2. 编写 docker-compose.yml 文件

   ```shell
   version: '3'
   services:
     nginx:
      image: nginx
      ports:
       - 80:80
      links:
       - app
      volumes:
       - ./nginx/conf.d:/etc/nginx/conf.d
     app:
       image: app
       expose:
         - "8080"
   ```

3. 创建./nginx/conf.d目录

   ```shell
   mkdir -p ./nginx/conf.d
   ```

4. 在./nginx/conf.d目录下 编写itheima.conf文件

   ```shell
   server {
       listen 80;
       access_log off;

       location / {
            proxy_pass http://app:8080;
       }
   
   }
   ```

5. 在~/docker-compose 目录下 使用docker-compose 启动容器

   ```shell
   docker-compose up
   ```

6. 测试访问

   ```shell
   http://192.168.149.135/hello
   ```



## Docker 私有仓库

### 搭建私有仓库

```shell
# 1、拉取私有仓库镜像 
docker pull registry
# 2、启动私有仓库容器 
docker run -id --name=registry -p 5000:5000 registry
# 3、打开浏览器 输入地址http://私有仓库服务器ip:5000/v2/_catalog，看到{"repositories":[]} 表示私有仓库 搭建成功
# 4、修改daemon.json   
vim /etc/docker/daemon.json    
# 在上述文件中添加一个key，保存退出。此步用于让 docker 信任私有仓库地址；注意将私有仓库服务器ip修改为自己私有仓库服务器真实ip 
{"insecure-registries":["私有仓库服务器ip:5000"]} 
# 5、重启docker 服务 
systemctl restart docker
docker start registry
```



### 将镜像上传至私有仓库

```shell
# 1、标记镜像为私有仓库的镜像     
docker tag centos:7 私有仓库服务器IP:5000/centos:7
 
# 2、上传标记的镜像     
docker push 私有仓库服务器IP:5000/centos:7
```



### 从私有仓库拉取镜像 

```shell
#拉取镜像 
docker pull 私有仓库服务器ip:5000/centos:7
```



## docker容器虚拟化 与 传统虚拟机比较

![image-20211219225925564](https://gitee.com/L-rw/img/raw/master/image-20211219225925564.png)

