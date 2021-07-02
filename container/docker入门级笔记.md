# 什么是docker？
使用Google公司推出的Go语言进行开发实现，对进程进行封装隔离，属于操作系统层面的虚拟化技术。由于docker运行的进程的隔离性，因此也称其为容器。
然后，Docker 在容器的基础上，进行了进一步的封装，从文件系统、网络互联等等，极大的简化了容器的创建和维护。使得 Docker 技术比虚拟机技术更为轻便、快捷。

传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程，比如我们开发的软件程序；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

# 为什么要使用docker？

## 更高效的利用系统资源
由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销，Docker 对系统资源的利用率更高。无论是应用执行速度、内存损耗或者文件存储速度，都要比传统虚拟机技术更高效。因此，相比虚拟机技术，一个相同配置的主机，往往可以运行更多数量的应用。

## 更快速的启动时间
Docker 容器应用，由于直接运行于宿主内核，无需启动完整的操作系统，因此可以做到秒级、甚至毫秒级的启动时间。

## 一致的运行环境
在开发过程中，使用docker有效的解决开发、测试、生产的环境混乱的问题。

## 持续交付和部署
使用docker可以结合dockerfile使构建、部署项目变的更容易，通过dockerfile开发人员可以更清楚的理解镜像build和run的过程，降低运维的成本，甚至，可以结合持续部署完成代码更新系统自动部署。

## 更轻松的迁移
既然docker能够确保环境的一致性，对于项目迁移就变得很简单、透明。它就像一个盒子，你把它放在桌子上和椅子上它还是那个盒子，仅此而已。

## 更容易扩展
通过docker的镜像技术，更容易被复用且在原有基础上做定制，甚至可以使他成为众多容器组件中的一员，这对应用程序的扩展非常有利。


# docker有哪些优势？和虚拟机区别是什么？
| 特性       | 容器               | 虚拟机     |
| ---------- | ------------------ | ---------- |
| 启动       | 秒级               | 分钟级     |
| 硬盘使用   | 一般为 MB          | 一般为 GB  |
| 性能       | 接近原生           | 弱于       |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |


# 结构
## 镜像
可以理解为docker的构建输出物就是镜像，比如一个java工程项目，通过Dockerfile中的一些build指令之后，可以在本地产生docker镜像，镜像时很独立的一套东西，容器的运行是完全基于镜像的。

## 容器
容易就是运行时的镜像了，在运行镜像时，可以输出更灵活的指令来使相同的镜像工作，最后通过docker管理，这些就是容器了，每个容器都是一个单独的应用，互相隔离。

## 注册中心
这个概念，有点像github，后者现在我们使用的码云，gitee代码托管库，试想一下，如果镜像都在每个人自己来管理，那会增加很多重复的工作，docker生态提供了注册中心，registry，也就是类似于dockerhub这一类云端管理服务，通过注册中心，我们可以将本地的镜像输出在远程，从而可以实现开发人员以及运维人员对版本、服务管理的灵活性和易操作性。

# 安装

官网安装步骤推荐：[https://docs.docker.com/engine/install/centos/#install-using-the-repository](https://docs.docker.com/engine/install/centos/#install-using-the-repository)

# 常用命令

```
systemctl start docker   启动docker,命令:
dockers version   验证docker是否启动成功,命令:
systemctl restart docker   重启docker,命令:
systemctl stop docker END   关闭docker
docker run id/name  
docker container prune 清理掉所有处于终止状态的容器
docker ps //查看正在运行的容器  
docker ps -a // 查看所有容器状态  
docker rm id/name 删除容器  
docker rmi id/name 删除某个镜像  
docker start/stop id/name 启动/停止某个容器  
docker attach id 进入某个容器(使用exit退出后容器也跟着停止运行)  
docker exec -ti id 启动一个伪终端以交互式的方式进入某个容器（使用exit退出后容器不停止运行）  
docker images 查看本地镜像  
docker run --name test -ti ubuntu /bin/bash 复制ubuntu容器并且重命名为test且运行，然后以伪终端交互式方式进入容器，运行bash  
docker build -t soar/centos:7.1 .  通过当前目录下的Dockerfile创建一个名为soar/centos:7.1的镜像   
docker run -d -p 2222:22 --name test soar/centos:7.1 以镜像soar/centos:7.1创建名为test的容器，并以后台模式运行，并做端口映射到宿主机2222端口，P参数重启容器宿主机端口会发生改变
docker run -i -t 容器 /bin/bash -c 'while true; do echo hello world; sleep 1; done' 运行容器 并且使用终端进行连接
```