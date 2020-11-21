# 简介

Docker是一个能够把开发的应用程序自动部署到容器的开源引擎。

##Docker特点

- Docker提供了一个简单、轻量的建模方式，大部分Docker容器只需要不到一秒钟就可以启动。
- Docker缩短了代码从开发测试到部署、上线运行的周期。
- Docker鼓励面向服务的架构和微服务架构，推荐单个容器只运行一个应用程序或进程。

## Docker组件

###Docker客户端和服务器

Docker是一个C/S架构的程序

//todo 

###Docker镜像

用户基于镜像来运行自己的容器；镜像是基于联合(Union)文件系统的一种层式的结构；镜像是由一系列指令一步步构建出来的；

本地镜像都保存在宿主机的/var/lib/docker目录下



###Registry

用来保存用户构建的镜像，分为公共和私有

###Docker容器

//todo 

## 安装Docker

**前提条件**

- 64位CPU的计算机

- Linux3.8或更高版本内核

# 使用前优化

## docker换国内源

换国内源按以下步骤即可

```shell
#打开一个配置文件
sudo vim /etc/docker/daemon.json
#写入以下信息
{
 "registry-mirrors": ["https://registry.docker-cn.com"]
}
#重启docker
sudo service docker restart
```



# 使用Docker

**查看容器信息**

使用`docker info` 确保docker已经就绪

`hostname` ：检查容器的主机名，即容器ID

`ps -aux`：检查容器的进程

`apt-get update && apt-get install vim` 	在容器中安装vim

`docker ps ` 	列出正在运行的容器

`docker ps -a` 	列出所有容器

**容器的创建和启动**

`docker run -i -t ubuntu /bin/bash`  创建一个容器并启动

- `-i` ：保证STDIN开启，以开启交互式shell
- `-t` ：为容器分配一个伪终端
- `ubuntu`：告诉Docker基于什么镜像来创建容器
- `/bin/bash`：在新容器中运行什么命令(运行/bin/shell命令启动了一个Bash shell)

`docker start/restart container-name/container-ID ` 	启动已经停止运行的容器

##常用命令

**镜像命令**

`docker search name` 	按名称查找镜像

`docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]`  	获取镜像

`docker build [选项] <上下文路径/URL/->`    	构建镜像  

- `[选项] -t`     设置镜像的名字及标签(-t image_name:version)

`docker image ls` 	列出镜像，只会显示顶层镜像

`docker image ls -a` 	显示包括中间层镜像在内的所有镜像

`docker image rm [选项] <镜像1> [<镜像2> ...] ` 	删除镜像

`docker rmi $(docker images -q -f dangling=true)` 	删除所有虚悬镜像

**容器启动/运行命令**

`docker run [选项] [镜像名] [命令]`     #就是运行容器的命令

- [选项] `-it`   	-i：交互式操作，一个是 -t 终端
- [选项] `--rm `      这个参数是说容器退出后随之将其删除
- [选项] ` -d`          让 Docker 在后台运行
- [选项] `-p 宿主机端口:容器端口`     端口映射(-p 8080:80) 
- [选项] `--name`      为容器指定一个名称(--name myName)
- [选项] `--restart=always`        自动重启容器(=on-failure:5容器退出代码为非0时重启，最多重启5次)
- [镜像名]     告诉Docker基于什么镜像来创建容器
- [命令] `bash`      以交互式 Shell操作

`docker stop container_name/container_id`     停止正在运行的容器
`docker start/restart container_name/container_id`      重新启动已经停止的容器

**查看容器信息**

`docker ls`   	查看容器信息
`docker logs [container ID or NAMES]`    	获取容器的输出信息

`docker system df`   	查看镜像、容器、数据卷所占用的空间
`docker stats`   	显示该主机上的一组容器的性能和指标

`docker inspect container_name  [container_name ...]` 	查看容器的详细信息及配置 

**删除容器**

`docker  rm [container_name]`    	来删除一个处于终止状态的容器
`docker prune`   	清理所有处于终止状态的容器

## 使用Dockerfile构建镜像

Dockerfile使用基本的基于DSL(Domain Specific Language)语法的指令来构建一个Docker镜像，使用Dockerfile构建的进行更具备可重复性、透明性、以及幂等性。

**示例：**

```shell
#Dockerfile支持注释
FROM ubuntu:14.04	#指定基础镜像
MAINTAINER James Tturnbull "james@example.com"	#指定维护者信息
RUN apt-get update && aptget install -y nginx 	#
RUN echo 'Hi,I am in your container'
EXPOSE 80
```

`docker run [参数] [容器名] [命令]`		容器启动命令

**RUN指令**

Dockerfile中的指令会按顺序从上到下执行，每条RUN指令都会创建一个新的镜像层并提交，然后基于刚提交的镜像执行下一条命令。

默认情况下，RUN指令会在shell里使用命令包装器/bin/sh -c来执行。如果不支持或不希望在shell中运行，也可以使用exec格式的RUN指令，这种方式中使用一个数组来指定要运行的命令和传递给该命令的每个参数。

`RUN ["apt-get","install","-y","nginx"]` 

执行docker build命令时，Dockerfile中的所有指令都会被执行并提交，并在该命令成功结束后返回一个新镜像。

RUN指令：指定镜像被构架时要运行的命令。

**CMD指令**

指定容器被启动时要运行的命令。docker会在指定的命令前加上/bin/sh -c

Dockerfile中只能有一条CMD命令，指定多条只有最后一条生效

docker run 命令可以覆盖CMD命令

```shell
docker run -i -t myUbuntu /bin/true
CMD ["/bin/true"]		#这两句等价
```

**ENTRYPOINT**

与CMD指令类似，但是不容易在启动时被覆盖。

通过数组的方式为为命令指定相应的参数`ENTRYPOINT ["/usr/sbin/nginx", "-g", "deamon off"]` 

```shell
#Dockerfile
...
ENTRYPOINT ["/usr/sbin/nginx"]
```

使用`docker run -t -i 容器名 -g "deamon off;"` 启动。指定 `-g "deamon off;"` 参数，这个参数会传递给用ENTRYPOINT指定的命令，构成`/usr/sbin/nginx -g "deamon off;"` 命令，以前台运行的方式启动Nginx守护进程。

**同时使用CMD和ENTRYPOINT命令**

组合使用CMD和ENTRYPOINT命令可以完成一些巧妙的工作

```shell
#Dockerfile
...
ENTRYPOINT ["/usr/sbin/nginx"]
CMD ["-h"]
```

使用`docker run `指定的命令参数都会被传递给ENTRYPOINT运行，如果不指定任何参数，则CMD指令中的会被传递，即以`/usr/sbin/nginx -h`方式启动。

**WORKDIR**

该指令用来从镜像创建一个新容器时，在容器内部设置一个工作目录，ENTRYPOINT和CMD指定的程序会在这个目录下执行

**ENV**

用来在镜像构建过程中设置环境变量

示例：

```shell
ENV RVM_PATH /home/rvm/
ENV RVM_PATH=/home/rvm/ RVM_ARCHFLAGS="-arch i386"
```

 这样的环境变量可以在后续的任何RUN指令中使用

也可以使用`docker run -e "RVM_PATH=/home/rvm/"` 来传递环境变量，这些变量只会在运行时有效

**USER**

该指令指定该镜像会以什么样的用户去运行

**VOLUME**

用来向基于镜像创建的容器添加卷，提供数据共享或持久化功能。卷可以在容器键共享和重用，对卷的修改立即生效

```shell
VOLUME ["/opt/project"]
#这条指令会为基于此镜像创建的任何容器创建一个名为/opt/project的挂载点；VOLUME ["v1", "v2"]指定多个卷
```

**ADD**

`ADD 源文件 目标文件/目录` 	ADD指令用来将构建环境下的文件和目录复制到镜像中

```shell
ADD software.lic /opt/app/software.lic
```

这里的ADD指令会将构建目录下的software.lic文件复制到镜像中的/opt/app/sofware.lic，

- 不能对**构建目录**或者**上下文**之外的文件进行ADD操作
- ADD指令会自动分辨目标是文件还是目录
- 若不存在文件/目录，则会被自动创建
- 如果源文件是一个压缩文件，会将其自动解压
- ADD指令会使构建缓存变得无效

**COPY**

和ADD指令功能相同，只是不会做文件的提取和解压工作

**LABEL**

用于为Docker镜像添加元数据，元数据以键值对的形式展现

```shell
LABEL version="1.0"		#指定一个元数据
LABEL location="New York" type="Data Center"		#指定多个元数据
```







## Dockerfile和构建缓存

由于每一步的构建过程都会将结果提交为镜像，docker会将之前的镜像层看做缓存。再次构建时，直接从上一步开始

```shell
FROM ubuntu:14.04
MAINTAINER James Turnbull "james@example.com"
ENV REFRESHED_AT 2020-05-05
RUN apt-get -qq update
```

使用ENV来在镜像中设置了一个名为REFRESHED_AT环境变量，这个环境变量用来表明该镜像模板最后的更新时间