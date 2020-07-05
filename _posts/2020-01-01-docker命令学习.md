

# Docker命令学习

**帮助目录**

[TOC]

## 1.Docker

本质：容器，解决端口冲突等。对服务进行打包隔离。



## 2.Docker相关网站

Docker基于Go语言开发

官网：https://www.docker.com/

Docker文档地址：https://docs.docker.com/

仓库地址：https://hub.docker.com/

## 3.容器化技术与传统虚拟机的对比

1. 传统虚拟机运行一个完整的操作系统，在系统上安装和运行软件。
2. 容器内应用直接运行在宿主机内核，容器本身无内核。
3. 每个容器间相互隔离，都有属于自己的文件系统，互相间不影响彼此。

## 4.Docker的基本组成

1. 镜像（image）：

   镜像就好比是一个模板，通过镜像来创建容器服务；通过镜像可以创建多个容器（服务或者项目运行的地方）。

2. 容器（container）：

   利用容器技术，独立运行一个或者一组应用，通过镜像来创建。

   基本命令：创建，停止，删除等。

3. 仓库（repository）：

   仓库就是存放镜像的地方。

   分为公有仓库和私有仓库。

   Docker Hub国外。

   配置镜像加速。

## 5.安装Docker（基于CentOS7）

``` 
环境准备: CentOS 7；Xshell
```

```shell
# 1、卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

```sh
# 2、需要的安装包
yum install -y yum-utils
```

```sh
# 3、设置镜像仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo #默认是国外的
	http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  #阿里云docker镜像源（推荐使用）
```

```sh
# 4、更新软件包索引
yum makecache fast
```

```sh
# 5、安装docker docker-ce 社区  ee 企业版
yum install docker-ce docker-ce-cli containerd.io

# 指定版本安装
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

```sh
# 6、启动docker

systemctl start docker

# 查看版本

docker version
```

```sh
# 7、跑hello world
docker run hello-world

# 查看hello world镜像
docker images
```

```sh
# 8、卸载docker
yum remove docker-ce docker-ce-cli containerd.io #卸载依赖

rm -rf /var/lib/docker #删除环境  “/var/lib/docker”-----默认工作路径
```

## 6.阿里云镜像加速

***前提：使用阿里云服务器并安装docker***

1、登陆阿里云，容器服务

https://account.aliyun.com/login/login.htm

2、阿里云镜像加速器

3、配置使用

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [" #阿里云提供给你的加速器地址# "]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 7.Docker命令



### 帮助命令

```sh
docker version  # --显示docker版本信息
docker info # --显示docker基本信息，包括客户端、服务器基本信息等
docker 命令 --help # --帮助命令
```

命令地址文档：https://docs.docker.com/reference/



### 镜像命令

**docker images   --查看所有本地主机上的镜像**

```sh
# REPOSITORY 镜像的仓库源
# TAG 镜像的标签
# IMAGE ID 镜像id
# CREATED 镜像创建时间
# SIZE 镜像的大小
```

**docker search  --搜索镜像**

```sh
# 可选项
# 1.通过收藏过滤
--filter=STARS=3000 #搜索点赞3000以上的镜像
# 使用
docker search mysql --filter=STARS=3000
```

**docker pull --下载镜像**

```sh
# 下载镜像 docker pull 镜像名[:TAG]

# 使用
docker pull mysql

# 关于
# 如果不适用TAG，默认安装latest版本
# 下载是分层下载 联合文件系统
```

**docker rmi --删除镜像**

```sh
docker rmi -f + 镜像id # 删除指定镜像 /加空格可删除多个
docker rmi -f $(docker images -aq) # 删除所有镜像
```



### 容器命令

***说明：有了镜像才可以创建容器，此处用centos镜像进行解释***

**先下载centos镜像**

```sh
docker pull centos
```

**新建容器并启动**

```sh
docker run 命令
docker run [可选参数] image
# 参数说明
--name="NAME" #容器名字

-d #后台运行，类似nohup

-it #进入容器，交互运行
# 使用
docker run -it centos /bin/bash #启动并进入容器，使用bash命令对二级系统进行控制（只是个基础版本，并不完善）
exit #退出容器

-p #指定容器端口（小写）
	-p ip：主机端口：容器端口
	-p 主机端口：容器端口（常用）
	-p 容器端口
-P #随机端口运行（大写）

# 使用
docker run centos
```

**列出所有运行的容器**

```sh
docker ps 命令
#参数说明
-a #列出当前正在运行的容器/带出历史运行的容器
-n=? #显示最近创建的容器，后为数字
-q #只显示容器编号
```

**退出容器**

```sh
exit #退出容器，并停止运行
ctrl +p +q #容器不停止退出
```

**删除容器**

```sh
docker rm 容器id   #删除指定容器，不能删除正在运行容器，需加-f
docker rm -f $(docker ps -aq) #删除所有容器
docker ps -a -q | xargs docker rm #删除所有容器
```

**启动和停止容器**

```shell
docker start 容器id # 启动容器
docker restart 容器id # 重启容器
docker stop 容器id # 停止当前正在运行的容器
docker kill 容器id # 强制停止当前正在运行的容器
```

### docker使用常见命令

**后台启动容器**

```sh
docker run -d 镜像名

#坑
#容器使用后台运行时，必须关联一个前台进程，若没有就会自动停止
```

**查看日志**

```sh
docker logs

docker logs -tf tail [nums] 容器id #nums显示的条数
```

**查看容器进程信息**

```sh
docker top 容器id
```

**查看镜像源数据**

```sh
docker inspect 容器id
```

**进入当前正在运行的容器**

```sh
#方式一
docker exec -it 容器id /bin/bash

#方式二
docker attach 容器id

###区别
# exec 进入容器后开启一个新的终端
# attach 进入正在执行的终端，不会启动新的终端
```

**从容器内拷贝至主机**

```sh
docker cp 容器id:容器文件路径 主机路径  #将容器内的文件拷贝出到主机上的路径
```

## **8.Docker命令文档**

```sh
attach      Attach local standard input, output, and error streams to a running container
build       Build an image from a Dockerfile
commit      Create a new image from a container's changes
cp          Copy files/folders between a container and the local filesystem
create      Create a new container
diff        Inspect changes to files or directories on a container's filesystem
events      Get real time events from the server
exec        Run a command in a running container
export      Export a container's filesystem as a tar archive
history     Show the history of an image
images      List images
import      Import the contents from a tarball to create a filesystem image
info        Display system-wide information
inspect     Return low-level information on Docker objects
kill        Kill one or more running containers
load        Load an image from a tar archive or STDIN
login       Log in to a Docker registry
logout      Log out from a Docker registry
logs        Fetch the logs of a container
pause       Pause all processes within one or more containers
port        List port mappings or a specific mapping for the container
ps          List containers
pull        Pull an image or a repository from a registry
push        Push an image or a repository to a registry
rename      Rename a container
restart     Restart one or more containers
rm          Remove one or more containers
rmi         Remove one or more images
run         Run a command in a new container
save        Save one or more images to a tar archive (streamed to STDOUT by default)
search      Search the Docker Hub for images
start       Start one or more stopped containers
stats       Display a live stream of container(s) resource usage statistics
stop        Stop one or more running containers
tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
top         Display the running processes of a container
unpause     Unpause all processes within one or more containers
update      Update configuration of one or more containers
version     Show the Docker version information
wait        Block until one or more containers stop, then print their exit codes

```

