# 盘它！一万多字，两小时带你入门 Docker ！（转载）

[TOC]

> 原文地址：
>
> 作者：Warm and new
>
> cnblogs.com/peng104/p/10296717.html

## 1、引言

### 1.1 Docker是什么

Docker 最初是 dotCloud 公司创始人 Solomon Hykes 在法国期间发起的一个公司内部项目，于 2013 年 3 月以 Apache 2.0 授权协议开源，主要项目代码在 GitHub 上进行维护。

Docker 使用 Google 公司推出的 Go 语言 进行开发实现。

docker是linux容器的一种封装，提供简单易用的容器使用接口。它是最流行的Linux容器解决方案。

docker的接口相当简单，用户可以方便的创建、销毁容器。

docker将应用程序与程序的依赖，打包在一个文件里面。运行这个文件就会生成一个虚拟容器。

程序运行在虚拟容器里，如同在真实物理机上运行一样，有了docker，就不用担心环境问题了。

### 1.2 应用场景

web应用的自动化打包和发布

自动化测试和持续集成、发布

在服务型环境中部署和调整数据库或其他应用

### 1.3 区别

1、物理机

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520145917.jpg)

2、虚拟机

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520145919.webp)

3、docker容器

![img](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520145920.jpg)

### 1.4 Docker的三大概念及优势

- 镜像　image
- 容器　container
- 仓库　repository

**docker容器的优势**

**1、更高效的利用系统资源**

由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销，Docker 对系统 资源的利用率更高。

无论是应用执行速度、内存损耗或者文件存储速度，都要比传 统虚拟机技术更高效。因此，相比虚拟机技术，一个相同配置的主机，往往可以运 行更多数量的应用。

**2、更快速的启动时间**

传统的虚拟机技术启动应用服务往往需要数分钟，而 Docker 容器应用，由于直接 运行于宿主内核，无需启动完整的操作系统，因此可以做到秒级、甚至毫秒级的启 动时间。大大的节约了开发、测试、部署的时间。

**3、一致的运行环境**

开发过程中一个常见的问题是环境一致性问题。由于开发环境、测试环境、生产环 境不一致，导致有些 bug 并未在开发过程中被发现。

而 Docker 的镜像提供了除内 核外完整的运行时环境，确保了应用运行环境一致性，从而不会再出现 “这段代码 在我机器上没问题啊” 这类问题。

**4、持续交付和部署**

对开发和运维(DevOps)人员来说，最希望的就是一次创建或配置，可以在任意 地方正常运行。

使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。开发人员 可以通过 Dockerfile 来进行镜像构建，并结合持续集成(Continuous Integration) 系 统进行集成测试，而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合 持续部署(Continuous Delivery/Deployment) 系统进行自动部署。

而且使用 Dockerfile 使镜像构建透明化，不仅仅开发团队可以理解应用运行环 境，也方便运维团队理解应用运行所需条件，帮助更好的生产环境中部署该镜像。

**5、更轻松的迁移**

由于 Docker 确保了执行环境的一致性，使得应用的迁移更加容易。Docker 可以在 很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是笔记本，其运 行结果是一致的。

因此用户可以很轻易的将在一个平台上运行的应用，迁移到另一 个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。

------

## 2、Docker安装

系统环境：docker最低支持centos7且在64位平台上，内核版本在3.10以上

版本：社区版，企业版（包含了一些收费服务）

官方版安装教程（英文）

> https://docs.docker.com/install/linux/docker-ce/centos/#upgrade-docker-after-using-the-convenience-script

博主版安装教程：

```shell
# 安装docker
yum install docker
# 启动docker 
systemctl start/status docker 
# 查看docker启动状态
docker version 
```

**配置加速器**

简介：DaoCloud 加速器 是广受欢迎的 Docker 工具，解决了国内用户访问 Docker Hub 缓慢的问题。DaoCloud 加速器结合国内的 CDN 服务与协议层优化，成倍的提升了下载速度。

DaoCloud官网：

> https://www.daocloud.io/mirror#accelerator-doc

```shell
# 一条命令加速（记得重启docker）
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://95822026.m.daocloud.io
```

------

## 3、Docker基础命令

docker --help（中文注解）

```shell
Usage:
docker [OPTIONS] COMMAND [arg...]
       docker daemon [ --help | ... ]
       docker [ --help | -v | --version ]
A
self-sufficient runtime for containers.

Options:
  --config=~/.docker              Location of client config files  #客户端配置文件的位置
  -D, --debug=false               Enable debug mode  #启用Debug调试模式
  -H, --host=[]                   Daemon socket(s) to connect to  #守护进程的套接字（Socket）连接
  -h, --help=false                Print usage  #打印使用
  -l, --log-level=info            Set the logging level  #设置日志级别
  --tls=false                     Use TLS; implied by--tlsverify  #
  --tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA  #信任证书签名CA
  --tlscert=~/.docker/cert.pem    Path to TLS certificate file  #TLS证书文件路径
  --tlskey=~/.docker/key.pem      Path to TLS key file  #TLS密钥文件路径
  --tlsverify=false               Use TLS and verify the remote  #使用TLS验证远程
  -v, --version=false             Print version information and quit  #打印版本信息并退出

Commands:
    attach    Attach to a running container  #当前shell下attach连接指定运行镜像
    build     Build an image from a Dockerfile  #通过Dockerfile定制镜像
    commit    Create a new image from a container's changes  #提交当前容器为新的镜像
    cp    Copy files/folders from a container to a HOSTDIR or to STDOUT  #从容器中拷贝指定文件或者目录到宿主机中
    create    Create a new container  #创建一个新的容器，同run 但不启动容器
    diff    Inspect changes on a container's filesystem  #查看docker容器变化
    events    Get real time events from the server#从docker服务获取容器实时事件
    exec    Run a command in a running container#在已存在的容器上运行命令
    export    Export a container's filesystem as a tar archive  #导出容器的内容流作为一个tar归档文件(对应import)
    history    Show the history of an image  #展示一个镜像形成历史
    images    List images  #列出系统当前镜像
    import    Import the contents from a tarball to create a filesystem image  #从tar包中的内容创建一个新的文件系统映像(对应export)
    info    Display system-wide information  #显示系统相关信息
    inspect    Return low-level information on a container or image  #查看容器详细信息
    kill    Kill a running container  #kill指定docker容器
    load    Load an image from a tar archive or STDIN  #从一个tar包中加载一个镜像(对应save)
    login    Register or log in to a Docker registry#注册或者登陆一个docker源服务器
    logout    Log out from a Docker registry  #从当前Docker registry退出
    logs    Fetch the logs of a container  #输出当前容器日志信息
    pause    Pause all processes within a container#暂停容器
    port    List port mappings or a specific mapping for the CONTAINER  #查看映射端口对应的容器内部源端口
    ps    List containers  #列出容器列表
    pull    Pull an image or a repository from a registry  #从docker镜像源服务器拉取指定镜像或者库镜像
    push    Push an image or a repository to a registry  #推送指定镜像或者库镜像至docker源服务器
    rename    Rename a container  #重命名容器
    restart    Restart a running container  #重启运行的容器
    rm    Remove one or more containers  #移除一个或者多个容器
    rmi    Remove one or more images  #移除一个或多个镜像(无容器使用该镜像才可以删除，否则需要删除相关容器才可以继续或者-f强制删除)
    run    Run a command in a new container  #创建一个新的容器并运行一个命令
    save    Save an image(s) to a tar archive#保存一个镜像为一个tar包(对应load)
    search    Search the Docker Hub for images  #在docker
hub中搜索镜像
    start    Start one or more stopped containers#启动容器
    stats    Display a live stream of container(s) resource usage statistics  #统计容器使用资源
    stop    Stop a running container  #停止容器
    tag         Tag an image into a repository  #给源中镜像打标签
    top       Display the running processes of a container #查看容器中运行的进程信息
    unpause    Unpause all processes within a container  #取消暂停容器
    version    Show the Docker version information#查看容器版本号
    wait         Block until a container stops, then print its exit code  #截取容器停止时的退出状态值

Run 'docker COMMAND --help' for more information on a command.  #运行docker命令在帮助可以获取更多信息
docker search  hello-docker  # 搜索hello-docker的镜像
docker search centos # 搜索centos镜像
docker pull hello-docker # 获取centos镜像
docker run  hello-world   #运行一个docker镜像，产生一个容器实例（也可以通过镜像id前三位运行）
docker image ls  # 查看本地所有镜像
docker images  # 查看docker镜像
docker image rmi hello-docker # 删除centos镜像
docker ps  #列出正在运行的容器（如果创建容器中没有进程正在运行，容器就会立即停止）
docker ps -a  # 列出所有运行过的容器记录
docker save centos > /opt/centos.tar.gz  # 导出docker镜像至本地
docker load < /opt/centos.tar.gz   #导入本地镜像到docker镜像库
docker stop  `docker ps -aq`  # 停止所有正在运行的容器
docker  rm `docker ps -aq`    # 一次性删除所有容器记录
docker rmi  `docker images -aq`   # 一次性删除所有本地的镜像记录
```

### 3.1 启动容器的两种方式

容器是运行应用程序的，所以必须得先有一个操作系统为基础

1、基于镜像新建一个容器并启动

```shell
# 1. 后台运行一个docker
docker run -d centos /bin/sh -c "while true;do echo 正在运行; sleep 1;done"
    # -d  后台运行容器
    # /bin/sh  指定使用centos的bash解释器
    # -c 运行一段shell命令
    # "while true;do echo 正在运行; sleep 1;done"  在linux后台，每秒中打印一次正在运行
docker ps  # 检查容器进程
docker  logs  -f  容器id/名称  # 不间断打印容器的日志信息 
docker stop centos  # 停止容器

# 2. 启动一个bash终端,允许用户进行交互
docker run --name mydocker -it centos /bin/bash  
    # --name  给容器定义一个名称
    # -i  让容器的标准输入保持打开
    # -t 让Docker分配一个伪终端,并绑定到容器的标准输入上
    # /bin/bash 指定docker容器，用shell解释器交互
```

当利用docker run来创建容器时，Docker在后台运行的步骤如下：

> 1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载
> 2. 利用镜像创建并启动一个容器
> 3. 分配一个文件系统，并在只读的镜像层外面挂在一层可读写层
> 4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
> 5. 从地址池配置一个ip地址给容器
> 6. 执行用户指定的应用程序
> 7. 执行完毕后容器被终止

2、将一个终止状态(stopped)的容器重新启动

```shell
[root@localhost ~]# docker ps -a  # 先查询记录
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS                    NAMES
ee92fcf6f32d        centos              "/bin/bash"              4 days ago          Exited (137) 3 days ago                                kickass_raman

[root@localhost ~]# docker start ee9  # 再启动这个容器
ee9

[root@localhost ~]# docker exec -it  ee9 /bin/bash  # 进入容器交互式界面
[root@ee92fcf6f32d /]#   # 注意看用户名，已经变成容器用户名
```

### 3.2 提交创建自定义镜像

```shell
# 1.我们进入交互式的centos容器中，发现没有vim命令
    docker run -it centos
# 2.在当前容器中，安装一个vim
    yum install -y vim
# 3.安装好vim之后，exit退出容器
    exit
# 4.查看刚才安装好vim的容器记录
    docker container ls -a
# 5.提交这个容器，创建新的image
    docker commit 059fdea031ba chaoyu/centos-vim
# 6.查看镜像文件
    docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
chaoyu/centos-vim   latest              fd2685ae25fe        5 minutes ago   
```

### 3.3 外部访问容器

容器中可以运行网络应用，但是要让外部也可以访问这些应用，可以通过-p或-P参数指定端口映射。

```shell
docker run -d -P training/webapp python app.py
  # -P 参数会随机映射端口到容器开放的网络端口

# 检查映射的端口
docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED            STATUS              PORTS                     NAMES
cfd632821d7a        training/webapp     "python app.py"     21 seconds ago      Up 20 seconds       0.0.0.0:32768->5000/tcp   brave_fermi
#宿主机ip:32768 映射容器的5000端口

# 查看容器日志信息
docker logs -f cfd  # #不间断显示log

# 也可以通过-p参数指定映射端口
docker run -d -p 9000:5000 training/webapp python app.py
```

打开浏览器访问服务器的9000端口， 内容显示 Hello world！表示正常启动

(如果访问失败的话，检查自己的防火墙，以及云服务器的安全组)

------

## 4、利用dockerfile定制镜像

镜像是容器的基础，每次执行docker run的时候都会指定哪个镜像作为容器运行的基础。我们之前的例子都是使用来自docker hub的镜像，直接使用这些镜像只能满足一定的需求，当镜像无法满足我们的需求时，就得自定制这些镜像。

> 镜像的定制就是定制每一层所添加的配置、文件。如果可以吧每一层修改、安装、构建、操作的命令都写入到一个脚本，用脚本来构建、定制镜像，这个脚本就是dockerfile。
>
> Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令 构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

参数详解

```shell
FROM scratch #制作base image 基础镜像，尽量使用官方的image作为base image
FROM centos #使用base image
FROM ubuntu:14.04 #带有tag的base image

LABEL version=“1.0” #容器元信息，帮助信息，Metadata，类似于代码注释
LABEL maintainer=“yc_uuu@163.com"

#对于复杂的RUN命令，避免无用的分层，多条命令用反斜线换行，合成一条命令！
RUN yum update && yum install -y vim 
    Python-dev #反斜线换行
RUN /bin/bash -c "source $HOME/.bashrc;echo $HOME”

WORKDIR /root #相当于linux的cd命令，改变目录，尽量使用绝对路径！！！不要用RUN cd
WORKDIR /test # 如果没有就自动创建
WORKDIR demo # 再进入demo文件夹
RUN pwd     # 打印结果应该是/test/demo

ADD and COPY 
ADD hello /  # 把本地文件添加到镜像中，吧本地的hello可执行文件拷贝到镜像的/目录
ADD test.tar.gz /  # 添加到根目录并解压

WORKDIR /root
ADD hello test/  # 进入/root/ 添加hello可执行命令到test目录下，也就是/root/test/hello 一个绝对路径
COPY hello test/  # 等同于上述ADD效果

ADD与COPY
   - 优先使用COPY命令
    -ADD除了COPY功能还有解压功能
添加远程文件/目录使用curl或wget

ENV # 环境变量，尽可能使用ENV增加可维护性
ENV MYSQL_VERSION 5.6 # 设置一个mysql常量
RUN yum install -y mysql-server=“${MYSQL_VERSION}” 
```

进阶知识(了解)

```shell
VOLUME and EXPOSE 
存储和网络

RUN and CMD and ENTRYPOINT
RUN：执行命令并创建新的Image Layer
CMD：设置容器启动后默认执行的命令和参数
ENTRYPOINT：设置容器启动时运行的命令

Shell格式和Exec格式
RUN yum install -y vim
CMD echo ”hello docker”
ENTRYPOINT echo “hello docker”

Exec格式
RUN [“apt-get”,”install”,”-y”,”vim”]
CMD [“/bin/echo”,”hello docker”]
ENTRYPOINT [“/bin/echo”,”hello docker”]


通过shell格式去运行命令，会读取$name指令，而exec格式是仅仅的执行一个命令，而不是shell指令
cat Dockerfile
    FROM centos
    ENV name Docker
    ENTRYPOINT [“/bin/echo”,”hello $name”]#这个仅仅是执行echo命令，读取不了shell变量
    ENTRYPOINT  [“/bin/bash”,”-c”,”echo hello $name"]

CMD
容器启动时默认执行的命令
如果docker run指定了其他命令(docker run -it [image] /bin/bash )，CMD命令被忽略
如果定义多个CMD，只有最后一个执行

ENTRYPOINT
让容器以应用程序或服务形式运行
不会被忽略，一定会执行
最佳实践：写一个shell脚本作为entrypoint
COPY docker-entrypoint.sh /usr/local/bin
ENTRYPOINT [“docker-entrypoint.sh]
EXPOSE 27017
CMD [“mongod”]

[root@master home]# more Dockerfile
FROm centos
ENV name Docker
#CMD ["/bin/bash","-c","echo hello $name"]
ENTRYPOINT ["/bin/bash","-c","echo hello $name”]
```

------

## 5、发布到仓库

### 5.1 docker hub共有镜像发布

docker提供了一个类似于github的仓库docker hub，官方网站（需注册使用）

> https://hub.docker.com/

```shell
# 注册docker id后，在linux中登录dockerhub
    docker login

# 注意要保证image的tag是账户名，如果镜像名字不对，需要改一下tag
    docker tag chaoyu/centos-vim peng104/centos-vim
    # 语法是：docker tag   仓库名   peng104/仓库名

# 推送docker image到dockerhub
    docker push peng104/centps-cmd-exec:latest

# 去dockerhub中检查镜像
# 先删除本地镜像，然后再测试下载pull 镜像文件
    docker pull peng104/centos-entrypoint-exec
```

### 5.2 私有仓库

docker hub 是公开的，其他人也是可以下载，并不安全，因此还可以使用docker registry官方提供的私有仓库

用法详解：

> https://yeasy.gitbooks.io/docker_practice/repository/registry.html

```shell
# 1.下载一个docker官方私有仓库镜像
    docker pull registry
# 2.运行一个docker私有容器仓库
docker run -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry  registry
    -d 后台运行 
    -p  端口映射 宿主机的5000:容器内的5000
    -v  数据卷挂载  宿主机的 /opt/data/registry :/var/lib/registry 
    registry  镜像名
    /var/lib/registry  存放私有仓库位置
# Docker 默认不允许非 HTTPS 方式推送镜像。我们可以通过 Docker 的配置选项来取消这个限制
# 3.修改docker的配置文件，让他支持http方式，上传私有镜像
    vim /etc/docker/daemon.json 
    # 写入如下内容
    {
        "registry-mirrors": ["http://f1361db2.m.daocloud.io"],
        "insecure-registries":["192.168.11.37:5000"]
    }
# 4.修改docker的服务配置文件
    vim /lib/systemd/system/docker.service
# 找到[service]这一代码区域块，写入如下参数
    [Service]
    EnvironmentFile=-/etc/docker/daemon.json
# 5.重新加载docker服务
    systemctl daemon-reload
# 6.重启docker服务
    systemctl restart docker
    # 注意:重启docker服务，所有的容器都会挂掉

# 7.修改本地镜像的tag标记，往自己的私有仓库推送
    docker tag docker.io/peng104/hello-world-docker 192.168.11.37:5000/peng-hello
    # 浏览器访问http://192.168.119.10:5000/v2/_catalog查看仓库
# 8.下载私有仓库的镜像
    docker pull 192.168.11.37:5000/peng-hello
```

------

## 6、实例演示

编写dockerfile，构建自己的镜像，运行flask程序。

确保app.py和dockerfile在同一个目录！

```shell
# 1.准备好app.py的flask程序
    [root@localhost ~]# cat app.py
    from flask import Flask
    app=Flask(__name__)
    @app.route('/')
    def hello():
        return "hello docker"
    if __name__=="__main__":
        app.run(host='0.0.0.0',port=8080)
    [root@master home]# ls
    app.py  Dockerfile

# 2.编写dockerfile
    [root@localhost ~]# cat Dockerfile
    FROM python:2.7
    LABEL maintainer="温而新"
    RUN pip install flask
    COPY app.py /app/
    WORKDIR /app
    EXPOSE 8080
    CMD ["python","app.py"]

# 3.构建镜像image,找到当前目录的Dockerfile，开始构建
    docker build -t peng104/flask-hello-docker .

# 4.查看创建好的images
    docker image ls

# 5.启动此flask-hello-docker容器，映射一个端口供外部访问
    docker run -d -p 8080:8080 peng104/flask-hello-docker

# 6.检查运行的容器
    docker container ls

# 7.推送这个镜像到私有仓库
    docker tag  peng104/flask-hello-docker   192.168.11.37:5000/peng-flaskweb
    docker push 192.168.11.37:5000/peng-flaskweb
```