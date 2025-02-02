---
redirect_from: /_posts/2023-01-24-2-Docker安装.md
title: Docker安装
tags: Docker
---

## 新建启动容器
```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
常用的参数：
- `--name`：为容器指定一个名称
- `-d`：后台运行容器并返回容器ID，也即启动守护式容器
- `-i`：以交互模式（interactive）运行容器，通常与-t同时使用
- `-t`：为容器重新分配一个伪输入终端（tty），通常与-i同时使用。也即启动交互式容器（前台有伪终端，等待交互）
- `-e`：为容器添加环境变量
- `-P`：随机端口映射。将容器内暴露的所有端口映射到宿主机随机端口
- `-p`：指定端口映射
`-p`指定端口映射的几种不同形式：
- `-p hostPort:containerPort`：端口映射，例如-p 8080:80
- `-p ip:hostPort:containerPort`：配置监听地址，例如 -p 10.0.0.1:8080:80
- `-p ip::containerPort`：随机分配端口，例如 -p 10.0.0.1::80
- `-p hostPort1:containerPort1 -p hostPort2:containerPort2`：指定多个端口映射，例如`-p 8080:80 -p 8888:3306`
### 启动交互式容器
```shell
# -i 交互模式
# -t 分配一个伪输入终端tty
# ubuntu 镜像名称
# /bin/bash（或者bash） shell交互的接口
docker run -it ubuntu /bin/bash
```
以交互方式启动ubuntu镜像
退出交互模式：
方式1：
```shell
# 在交互shell中exit即可退回宿主机
exit;
```
方式2：使用快捷键`ctrl` + `P` + `Q`
- 方式1 退出后，容器会停止；
- 方式2 退出后容器依然正在运行。
### 启动守护式容器
大部分情况下，我们系统docker容器服务时在后台运行的，可以通过`-d`指定容器的后台运行模式：
```shell
docker run -d 容器名
```
注意事项：
如果使用`docker run -d ubuntu`尝试启动守护式的ubuntu，会发现容器启动后就自动退出了。
因为Docker容器如果在后台运行，就必须要有一个前台进程。容器运行的命令如果不是那些一直挂起的命令（例如`top`、`tail`），就会自动退出。

## 列出正在运行的容器
列出所有正在运行的容器：
```shell
docker ps [OPTIONS]
```
常用参数：
- `-a`：列出当前所有正在运行的容器+历史上运行过的容器
- `-l`：显示最近创建的容器
- `-n`：显示最近n个创建的容器
- `-q`：静默模式，只显示容器编号

## 容器其他启停操作
### 启动已经停止的容器
```shell
docker start 容器ID或容器名
```
### 重启容器

```shell
docker restart 容器ID或容器名
```
### 停止容器

```shell
docker stop 容器ID或容器名
```
### 强制停止容器

```shell
docker kill 容器ID或容器名
```

## 删除容器

删除已经停止的容器：
```shell
docker rm 容器ID或容器名
```
删除容器是 `docker rm`，删除镜像是 `docker rmi`，注意区分。
强制删除正在运行的容器：
```shell
docker rm -f 容器ID或容器名
```
一次删除多个容器实例：
```shell
docker rm -f ${docker ps -a -q}

# 或者
docker ps -a -q | xargs docker rm
```
## 查看容器日志

```shell
docker logs 容器ID或容器名
```

## 查看容器内运行的进程

```shell
docker top 容器ID或容器名
```

## 查看容器内部细节

```shell
docker inspect 容器ID或容器名
```

## 进入正在运行的容器
进入正在运行的容器，并以命令行交互：

```shell
docker exec -it 容器ID bashShell
```

重新进入：

```shell
docker attach 容器ID
```

`docker exec` 和 `docker attach` 区别：
- `attach`直接进入容器启动命令的终端，不会启动新的进程，用exit退出会导致容器的停止
- `exec`是在容器中打开新的终端，并且可以启动新的进程，用exit退出不会导致容器的停止
如果有多个终端，都对同一个容器执行了 docker attach，就会出现类似投屏显示的效果。一个终端中输入输出的内容，在其他终端上也会同步的显示。
## 容器和宿主机文件拷贝

容器内文件拷贝到宿主机：

```shell
docker cp 容器ID:容器内路径 目的主机路径
```

宿主机文件拷贝到容器中：

```shell
docker cp 主机路径 容器ID:容器内路径
```

## 导入和导出容器

- `export`：导出容器的内容流作为一个tar归档文件（对应import命令）；
- `import`：从tar包中的内容创建一个新的文件系统再导入为镜像（对应export命令）；
  
示例：

```shell
# 导出
# docker export 容器ID > tar文件名
docker export abc > aaa.tar

# 导入
# cat tar文件 | docker import - 自定义镜像用户/自定义镜像名:自定义镜像版本号
docker aaa.tar | docker import - test/mytest:1.0.1
```

## 将容器生成新镜像

`docker commit`提交容器副本使之成为一个新的镜像。
> docker 启动一个镜像容器后， 可以在里面执行一些命令操作，然后使用`docker commit`将新的这个容器快照生成一个镜像。  

```shell
docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[tag]
```

Docker挂载主机目录，可能会出现报错：`cannot open directory .: Perission denied`。
解决方案：在命令中加入参数 `--privileged=true`。
CentOS7安全模块比之前系统版本加强，不安全的会先禁止，目录挂载的情况被默认为不安全的行为，在SELinux里面挂载目录被禁止掉了。如果要开启，一般使用 `--privileged=true`，扩大容器的权限解决挂载没有权限的问题。也即使用该参数，容器内的root才拥有真正的root权限，否则容器内的root只是外部的一个普通用户权限。

## 容器数据卷

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过UnionFS，提供一些用于持续存储或共享数据。

特性：卷设计的目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷。

特点：
- 数据卷可以在容器之间共享或重用数据
- 卷中的更改可以直接实施生效
- 数据卷中的更改不会包含在镜像的更新中
- 数据卷的生命周期一直持续到没有容器使用它为止

运行一个带有容器卷存储功能的容器实例：

```shell
docker run -it --privileged=true -v 宿主机绝对路径目录:容器内目录[rw | ro] 镜像名
```

可以使用`docker inspect`查看容器绑定的数据卷。

权限：
- `rw`：读写 
- `ro`：只读。如果宿主机写入内容，可以同步给容器内，容器内可以读取。 
- 
容器卷的继承：

```shell
# 启动一个容器
docker run -it --privileged=true /tmp/test:/tmp/docker --name u1 ubuntu /bin/bash

# 使用 --volumes-from 继承 u1的容器卷映射配置
docker run -it --privileged=true --volumes-from u1 --name u2 ubuntu
```

## 所有命令示意图
![](/assets/image/Docker/Docker安装/Docker命令示意图.png)