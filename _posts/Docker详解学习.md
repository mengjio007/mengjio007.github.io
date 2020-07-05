

# Docker详解学习

## 1.可视化（玩玩即可，基本不使用）

1.Portainer

```sh
docker run -d -p 9000:9000 \
--restart=always 
-v /var/run/docker.sock:/var/run/docker.sock 
docker.io/portainer/portainer
```

***Portainer介绍：***

Docker图形化界面管理工具！提供一个后台面板供操作！

## 2.Docker镜像详解

镜像是一种轻量级，可执行软件包，用于打包软件运行环境和基于运行环境开发的软件。包含运行一个软件需要的所有内容。

***镜像获取方式***

*1.远程仓库获取*

*2.获取别人dockerfile*

*3.自己写dockerfile*

### Docker镜像加载原理

```）sh
UnionFS（联合文件系统）
```

它可以把多个目录(也叫分支)内容联合挂载到同一个目录下，而目录的物理位置是分开的。UnionFS允许只读和可读写目录并存，就是说可同时删除和增加内容。**（分层管理）**

```sh
Docker镜像加载原理
```

docker 的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system) 主要包含bootloader和kernel，bootloader 主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就存在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

roorfs （root file system），在bootfs之上。包含的就是典型Linux系统中的 /dev ，/proc，/bin ，/etx 等标准的目录和文件。rootfs就是各种不同的操作系统发行版。比如Ubuntu，Centos等等。

对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host（宿主机）的kernel，自己只需要提供rootfs就行了，由此可见对于不同的Linux发行版，bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以公用bootfs。

​	***由于分层管理的机制，使得docker镜像可以在很小的情况下共用一套内核，逐渐叠加实现基本功能。（即平时几个G的系统，docker安装只需要几百M）***

```sh
分层理解
```

所有的Docker镜像都起始于一个最基础的镜像，当进行修改或增加内容时，就会在当前镜像层之上，创建新的镜像层。

*例：若在ubuntu创建一个镜像，就会是新镜像的第一层，当此时在此系统中在创建编程语言开发环境（例Java），就会在第一层的基础上创建第二层镜像；若继续添加补丁之类的则会在第二层上再创建第三层镜像。*

而外添加镜像层的同时，原镜像层并不改变，只是在此基础上整个镜像的叠加。

 ***docker 镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作 “容器层” ，“容器层” 之下的都叫镜像层。***

即自己pull下来镜像后的操作都是基于容器层，也就是最上层的操作。

### 提交自己的镜像 commit

```sh
docker commit #提交容器成为新的副本
#类似git命令
docker commit -m="描述信息" -a="作者" 容器id 目标镜像名TAG
```

## 3.容器数据卷

****

***需求：数据持久化***

****

***如mysql数据存储到本地而非只在容器中***

****

***即容器数据的持久化和同步！除此之外还有不同容器间数据共享！***

****

***容器数据卷***

当我们在使用docker容器的时候，会产生一系列的数据文件，这些数据文件在我们关闭docker容器时是会消失的，但是其中产生的部分内容我们是希望能够把它给保存起来另作用途的，Docker将应用与运行环境打包成容器发布，我们希望在运行过程钟产生的部分数据是可以持久化的的，而且容器之间我们希望能够实现数据共享。

通俗地来说，docker容器数据卷可以看成使我们生活中常用的u盘，它存在于一个或多个的容器中，由docker挂载到容器，但不属于联合文件系统，Docker不会在容器删除时删除其挂载的数据卷。

特点：

1：数据卷可以在容器之间共享或重用数据

2：数据卷中的更改可以直接生效

3：数据卷中的更改不会包含在镜像的更新中

4：数据卷的生命周期一直持续到没有容器使用它为止

***容器数据卷使用***

#### 挂载方式一

```sh
#方式一：通过命令挂载

#指定路径挂载

docker run -v 主机目录:容器内目录  # docker inspect 容器id --查看详细信息-------其中名为Mounts的是文件挂载的内容


# 示例挂载MySQL配置文件和数据目录

docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7 # d后台运行，p端口配置，v数据卷挂载，e环境配置（此处即mysql密码），--name更改镜像名

```

```sh
#匿名挂载
docker run -v 容器内目录
#示例
docker run -d -p --name nginx01 -v /etc/nginx nginx

#具名挂载
docker run -d -p --name aaa juming_nginx:/etc/nginx nginx
```

```sh
#docker volume 数据卷操作
docker volume create volume_name #表示创建一个数据卷。
docker volume ls #列出数据卷列表
docker volume rm volume_name #删除指定数据卷
docker volume inspect volume_name #查看数据卷的详细信息
```

容器内的卷，在没有目录的情况下，存在于     ***“/var/lib/docker/volume/xxx/_data”***

大多数情况皆使用具名挂载

```sh
拓展：
#通过-v 容器内路径：ro rw 设置读写权限
ro readonly 只读
rw readwrite 可读可写
#一旦设定权限，容器对挂载文件就会有限定。
示例：
docker run -d -P --name nginx02 -v juming_nginx:/etc/nginx:ro nginx 
docker run -d -P --name nginx02 -v juming_nginx:/etc/nginx:rw nginx
#有ro 则只能从容器外部（宿主机）对文件进行修改，容器内部无法操作
```

#### 挂载方式二

```sh
#方式二：通过dockerfile挂载

#编写简单dockerfile
vim dockerfile01
""
FROM centos

VOLUME ["volume01","volume02"] #匿名挂载

CMD echo "---end---"
CMD /bin/bash
""
docker build -f /dockerfile01路径 -t 镜像名:版本 . #最后一个点不要忘记
dockerfile中每一个命令就是一层（镜像分层）
```

#### 数据卷容器

两个或多个容器间进行数据共享

```sh
#创建父容器
docker run -it - -name parentContainer  镜像名（可以填上面通过dockerFile建立的镜像，里面有挂载容器卷）

#创建两个子容器   通过 --volumes -from 继承父容器
docker run -it - -name sonContainer1 --volumes-from parentContainer 镜像名

docker run -it - -name sonContainer2 --volumes-from parentContainer 镜像名
假设我们DockerFile里面定义的容器卷目录为dockerVolume，父容器里面有dockerVolume目录，子容器继承了父容器的dockerVolume，在字容器中的dockerVolume目录作出的修改会同步到父容器的dockerVolume目录上，达到了继承和数据共享的目的。
```

容器之间配置信息的传递，数据卷的生命周期会一致持续到没有容器使用它为止，换言之，只要有一个容器仍在使用该数据卷，该数据卷一直都可以进行数据共享，**通俗地来说，如果此时我们把父容器关闭掉，两个字容器之间依旧可以进行数据共享，而且通过继承子容器生成的新容器，一样可以与子容器进行数据共享**。这就是docker容器间的数据传递共享。

```sh
#示例：多个MySQL实现数据共享
#MySQL01
docker run -d -p 3306:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

#MySQL02
docker run -d -p 3306:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 mysql:5.7

#可以实现两个容器间数据共享
```



## 4.DockerFile详解和制作

dockerfile是用来构建docker镜像的文件！本身是一个命令参数脚本！

构建步骤：

1.编写一个dockerfile文件

2.dockerbuild构建成为一个镜像

3.docker run 运行镜像

4.docker push 发布镜像（DockerHub、阿里云镜像仓库）

**Dockerfile构建**

基础知识：

*每个保留关键字都必须大写；执行从上到下顺序执行；#表注释；每个指令都会创建提交一个新的镜像层，并提交。*

### DockerFIle命令

**FROM**

功能为指定基础镜像，并且必须是第一条指令。

如果不以任何镜像为基础，那么写法为：FROM scratch。

同时意味着接下来所写的指令将作为镜像的第一层开始

**LABEL**

功能是为镜像指定标签

**MAINTAINER**

指定作者

**EXPOSE**

功能为暴漏容器运行时的监听端口给外部

但是EXPOSE并不会使容器访问主机的端口

如果想使得容器与主机的端口有映射关系，必须在容器启动的时候加上 -P参数

**RUN**

 功能为运行指定的命令

RUN命令有两种格式

1. RUN <command>

2. RUN ["executable", "param1", "param2"]

   第一种后边直接跟shell命令

   在linux操作系统上默认 /bin/sh -c

   在windows操作系统上默认 cmd /S /C

   第二种是类似于函数调用。

   可将executable理解成为可执行文件，后面就是两个参

**ENV**

功能为设置环境变量

**CMD**

功能为容器启动时要运行的命令

RUN是构件容器时就运行的命令以及提交运行结果

CMD是容器启动时执行的命令，在构件时并不运行，构件时紧紧指定了这个命令到底是个什么样子

**ADD**

 一个复制命令，把文件复制到景象中。

如果把虚拟机与容器想象成两台linux服务器的话，那么这个命令就类似于scp，只是scp需要加用户名和密码的权限验证，而ADD不用。

**COPY**

看这个名字就知道，又是一个复制命令

与ADD的区别

COPY的<src>只能是本地文件，其他用法一致

**ENTRYPOINT**

功能是启动时的默认命令

 ```sh
#与CMD比较说明（这俩命令太像了，而且还可以配合使用）：

1. 相同点：

只能写一条，如果写了多条，那么只有最后一条生效

容器启动时才运行，运行时机相同

 

2. 不同点：

 ENTRYPOINT不会被运行的command覆盖，而CMD则会被覆盖

 如果我们在Dockerfile种同时写了ENTRYPOINT和CMD，并且CMD指令不是一个完整的可执行命令，那么CMD指定的内容将会作为ENTRYPOINT的参数
 ```



**VOLUME**

可实现挂载功能，可以将内地文件夹或者其他容器种得文件夹挂在到这个容器中

**USER**

设置启动容器的用户，可以是用户名或UID

**WORKDIR**

设置工作目录，对RUN,CMD,ENTRYPOINT,COPY,ADD生效。如果不存在则会创建，也可以设置多次。

**ARG**

设置变量命令，ARG命令定义了一个变量，在docker build创建镜像的时候，使用 --build-arg <varname>=<value>来指定参数

如果用户在build镜像时指定了一个参数没有定义在Dockerfile种，那么将有一个Warning

**ONBUILD**

**STOPSIGNAL**

STOPSIGNAL命令是的作用是当容器推出时给系统发送什么样的指令

**HEALTHCHECK**

 容器健康状况检查命令

## 5.Docker网络

每启动一个docker容器就会为一个容器分配一个内网ip

执行

```sh
ip addr #查看
```

会有3个地址：

**本机回环地址   lo，阿里云内网地址  eth0，docker0**

### ***docker0***

充当路由器功能

容器网卡都是一对一对，evth-pair技术，一端连接协议，一端彼此相连

通过evth-pair充当桥梁，连接虚拟网络设备

容器间可以相互ping通

删除容器后对应网桥删除

## ***--link***

原理：在hosts配置中增加了一个映射

## ***自定义网络***

**网络模式**

1.桥接模式：bridge，docker上搭桥（默认）

2.host：和宿主机共享网络

3.container：容器网络连通（容器之间直接互连，用的少，局限很大）

4.none：不配置网络

```sh
普通启动容器默认docker网络是 --net bridge
#默认；域名不能访问 可以--link打通
```

自定义网络( --help 查看所有可配置项)

```sh
docker network create --driver bridge --subnet 192.168.0.0/24 --gateway 192.168.0.1 mengjionet
```

自定义网络间可以用容器名ping通，修复了docker0的缺点。

***好处：***

***不同集群使用不同网络，相互间隔离***

### *网络联通*