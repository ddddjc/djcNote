# Docker

## 1.简介

主要解决了运行环境与配置问题的软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术

与虚拟机区别：

Docker容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，而传统虚拟机则是在硬件层面上实现虚拟化。与传统虚拟机此昂比，docker优势体现为启动速度快，占用体积小

## 2.概念层

### 镜像分层：

![image-20230813204430460](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308132044503.png)

1. 在拉取的时候会发现下载是下载了好几个内容，这实际上是docker分层的原因。

2. docker镜像层都是只读的，容器层都是可写的

3. Docker中的镜像分层支持通过拓展现有镜像。类似于java继承与一个Base基础类u，自己再按需拓展。

4. 新镜像是从base镜像一层一层叠加生成的。每安装一个软件，就在现有基础上增加一层
5. 这样分层的最大的一个好处是共享资源，方便迁移，就是为了复用

比如有许多镜像都是从相同的base镜像构建而来，那么Docker Host只需在磁盘上保存`一份base镜像`;同时内存中也只需加载一份base镜像，就可以为所有容器服务，而且镜像的每一层都可以**被**共享



![image-20230813210122802](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308132101884.png)

在上传的时候，有两个，其中一个大小和原版ubuntu相同，另一个约等于vim空间，可见是分层的。

### 容器数据卷：

> 经常使用，用于保存容器产生的数据，防止在删除容器后数据丢失

#### 概念：

卷就是目录或者文件，存在于一个活多个容器内，有Docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于程序存储活共享数据的特性：

卷的设计目的就是数据的持久化，完全独立于容器的生命周期，因此Docker不会在容器删除时删除器挂在的数据卷，相当于主机与镜像容器共享一块区域

#### 特点：

1. 可以在容器之间共享或重用数据
2. 卷中的更改可以直接实时生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到有没有容器使用它为止

#### 使用：

- docker run -it --privileged=true -v /宿主机绝对路径目录 :/容器内目录  镜像名
- docker run -it --privileged=true -v /宿主机绝对路径目录 :/容器内目录 re/ro 镜像名       通过设置读写权限，来实现不同的需求

#### 继承：了解

## 3.命令

### 1.帮助启动类命令

![image-20230813154042240](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308131540335.png)

### 2.镜像命令

- docker images [OPTIONS] [REPOSITORY[:TAG]]   ---列出本地自己上的镜像![image-20230813154754424](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308131547464.png)

  - **REPOSITORY：**表示镜像的仓库源
  - **TAG：**镜像的标签
  - **IMAGE ID：**镜像ID
  - **CREATED：**镜像创建时间
  - **SIZE：**镜像大小

  统一仓库源可以有多个TAG，代表这个仓库源不同版本。

-  docker pull [OPTIONS] NAME[:TAG|@DIGEST]   ---拉取镜像资源

- docker search [OPTIONS] TERM                             ---查找镜像

  ![image-20230813165817891](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308131658948.png)

  - **NAME:** 镜像仓库源的名称
  - **DESCRIPTION:** 镜像的描述
  - **OFFICIAL:** 是否 docker 官方发布
  - **stars:** 类似 Github 里面的 star，表示点赞、喜欢的意思。
  - **AUTOMATED:** 自动构建

- docker rmi [OPTIONS] IMAGE [IMAGE...]        ---删除镜像

- 创建镜像：

  - 当下载的镜像不能满足需求时，可以通过两种方式创建镜像

    - 从以常见的容器中更新镜像，并且提交这个镜像
    - 使用dockerfile指令来创建一个新的镜像

  - 更新镜像

    - 先使用镜像创建一个容器
    - 在容器内使用 apt-get update命令进行更新，进行其他操作，例如安装vim
    - 操作完后exit退出
    - 通过docker commit来提交容器副本
    - docker commit -m='hasvim' -a='djc' 1e5c794076a3 djcubuntu:1.1
    - 各个参数说明：
      - **-m:** 提交的描述信息
      - **-a:** 指定镜像作者
      - **e218edb10161：**容器 ID
      - **runoob/ubuntu:v2:** 指定要创建的目标镜像名
    
  - 构建镜像
  
    - 使用docker build，从零来创建一个新的镜像。为此，需要先创建一个Dockerfile文件，其中包括一组指令来告诉Docker如何构建镜像
  
      ```
      runoob@runoob:~$ cat Dockerfile 
      FROM    centos:6.7
      MAINTAINER      Fisher "fisher@sudops.com"
      
      RUN     /bin/echo 'root:123456' |chpasswd
      RUN     useradd runoob
      RUN     /bin/echo 'runoob:123456' |chpasswd
      RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
      EXPOSE  22
      EXPOSE  80
      CMD     /usr/sbin/sshd -D
      ```
  
    - 每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。
  
      第一条FROM，指定使用哪个镜像源
  
      RUN 指令告诉docker 在镜像内执行命令，安装了什么。。。
  
      然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。
  
      ```
      runoob@runoob:~$ docker build -t runoob/centos:6.7 .
      Sending build context to Docker daemon 17.92 kB
      Step 1 : FROM centos:6.7
       ---&gt; d95b5ca17cc3
      Step 2 : MAINTAINER Fisher "fisher@sudops.com"
       ---&gt; Using cache
       ---&gt; 0c92299c6f03
      Step 3 : RUN /bin/echo 'root:123456' |chpasswd
       ---&gt; Using cache
       ---&gt; 0397ce2fbd0a
      Step 4 : RUN useradd runoob
      ......
      ```
  
      参数说明：
  
      - **-t** ：指定要创建的目标镜像名
      - **.** ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径
  
- docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]          为镜像添加一个新的标签

- 

### 3.容器命令

- docker run [OPTIONS] IMAGE [COMMAND] [ARG...]  ---使用镜像启动（创建）一个容器（exit退出终端，-d后台运行）
- docker ps [OPTIONS]                                                        ----查看容器（-l,查询最后一次创建的容器）
- docker start [OPTIONS] CONTAINER [CONTAINER...]  ---启动一个或多个已经停止的容器
- docker stop [OPTIONS] CONTAINER [CONTAINER...]   ---停止一个容器的运行
- docker restart [OPTIONS] CONTAINER [CONTAINER...]  ---重启一个容器
- docker attach [OPTIONS] CONTAINER                       ---进入容器（后台），退出后容器停止
- docker exec [OPTIONS] CONTAINER COMMAND [ARG...]  ---进入容器（后台），推荐，退出后不会停止
- docker export [OPTIONS] CONTAINER                            ---到出本地某个容器
- docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]  ---将容器快照文件导入到镜像
- docker rm [OPTIONS] CONTAINER [CONTAINER...]      ---删除容器 
- docker container prune                                                          ---删除所有处于终止状态的容器
- `docker image prune`                                                            ---删除所有虚悬镜像
- docker run -d -p 5000:5000 training/webapp python app.py  ---通过-p来设置端口
- docker port CONTAINER [PRIVATE_PORT[/PROTO]]       ---查看指定的ID或者名字容器的某个确定端口映射到主机的端口号
- docker logs [OPTIONS] CONTAINER                                   ---查看日志
- docker top CONTAINER [ps OPTIONS]                                 ---查看容器内部运行的进程
- docker inspect [OPTIONS] NAME|ID [NAME|ID...]           ---查看Docker底层的信息，返回一个JSON文件记录着Docker容器的配置和状态信息

# DockerDile

Dockerfile是用来构建Docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本

### 构建步骤：

- 编写Dockerfile文件
- docker build命令构建镜像
- docker run 以镜像运行容器实例

### Dockerfile指令特点：

1. 每条保留指令都`必须为大写字母`且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. #表示注释
4. 每条指令都会创建一个新的镜像层并对镜像进行提交

### Dockerfile指令

[mysql-Dockerfile参考](https://github.com/docker-library/mysql/blob/c13cda9c2c9d836af9b3331e8681f8cc8e0a7803/5.7/Dockerfile.oracle)

| Dockerfile 指令 | 说明                                                         |
| :-------------- | :----------------------------------------------------------- |
| FROM            | 指定基础镜像，用于后续的指令构建。                           |
| MAINTAINER      | 指定Dockerfile的作者/维护者。（已弃用，推荐使用LABEL指令）   |
| LABEL           | 添加镜像的元数据，使用键值对的形式。                         |
| RUN             | 在构建过程中在镜像中执行命令。                               |
| CMD             | 指定容器创建时的默认命令。（可以被覆盖）                     |
| ENTRYPOINT      | 设置容器创建时的主要命令。（不可被覆盖）                     |
| EXPOSE          | 声明容器运行时监听的特定网络端口。                           |
| ENV             | 在容器内部设置环境变量。                                     |
| ADD             | 将文件、目录或远程URL复制到镜像中。                          |
| COPY            | 将文件或目录复制到镜像中。                                   |
| VOLUME          | 为容器创建挂载点或声明卷。                                   |
| WORKDIR         | 设置后续指令的工作目录。（进入终端后的落脚点）               |
| USER            | 指定后续指令的用户上下文。                                   |
| ARG             | 定义在构建过程中传递给构建器的变量，可使用 "docker build" 命令设置。 |
| ONBUILD         | 当该镜像被用作另一个构建过程的基础时，添加触发器。           |
| STOPSIGNAL      | 设置发送给容器以退出的系统调用信号。                         |
| HEALTHCHECK     | 定义周期性检查容器健康状态的命令。                           |
| SHELL           | 覆盖Docker中默认的shell，用于RUN、CMD和ENTRYPOINT指令。      |

参考[Docker Dockerfile | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-dockerfile.html)

### docker执行Dockerfile的大致流程：

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行docker 中的下一条指令直到所有指令都执行完成



### docker微服务：

1. 打包jar包
2. 上传到服务器
3. 编写Dockerfile，引导入ar包以及启动
4. build镜像

### docker网络

![image-20230814143042198](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308141430712.png)

类似于虚拟机都有一个网络连接模式，docker的每个容器也都有各自的网络连接模式（docker network ls)

![image-20230814143222044](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308141432939.png)

**功能：**

- 容器间的互联和通信以及端口映射
- 容器ip变动时可以通过服务名直接网络通信而不受影响

| 网络模式  | 简介                                                         |
| --------- | ------------------------------------------------------------ |
| bridge    | 为每个容器分配、设置IP等，并将容器链接到·docker0，虚拟网桥，默认为该模式 |
| host      | 容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口 |
| none      | 容器有独立的network namespace，但并没有对其进行任何网络设置，如分配veth pair 和网桥连接，ip等 |
| container | 新创建的容器不会创建自己的网卡和配置自己的IP，而是和一个指定的容器共享IP，端口范围等。 |

自定义网络：

docker network create djcnetwork

![image-20230814151122748](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308141511795.png)

`docker run -d -p 8081:8080 --network djcnetwork --name djctomcat1 tomcat`

--network 指定运行哪个网络。

### docker compose

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。写好多个容器之间的调用关系，然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

Compose使用的步骤

1. 使用Dockerfile定义应用程序环境
2. 使用docker-compose.yml定义构成应用程序的服务，这样它可以在隔离环境中一起运行
3. 最后，执行docker-compose up 命令来启动并运行整个应用程序

