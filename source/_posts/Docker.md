---
title: Docker简介
date: 2019-03-09 21:39:08
tags: Docker
---
一句话概括容器：容器就是将软件打包成标准化单元，以用于开发、交付和部署。
* 容器镜像是轻量的、可执行的独立软件包 ，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
* 容器化软件适用于基于Linux和Windows的应用，在任何环境中都能够始终如一地运行。
* 容器赋予了软件独立性　，使其免受外在环境差异（例如，开发和预演环境的差异）的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。
<!--more-->
![image](http://490.github.io/images/20190311_082055.png)
拿内存举例，虚拟机是利用Hypervisor去虚拟化内存，整个调用过程是虚拟内存->虚拟物理内存->真正物理内存，但是Docker是利用Docker Engine去调用宿主的的资源，这时候过程是虚拟内存->真正物理内存。

传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便.
**虚拟机更擅长于彻底隔离整个运行环境**。例如，云服务提供商通常采用虚拟机技术隔离不同的用户。而 **Docker通常用于隔离不同的应用** ，例如前端，后端以及数据库。
![image](http://490.github.io/images/20190311_081729.png)
# Docker 引擎

Docker 是一个典型的 C/S 模式的架构，后端是一个松耦合架构，模块各司其职。

用户使用 Docker Client 与 Docker Daemon 建立通信，并发送请求给后者。

以前的 Docker Daemon 超级的万能，基本什么都是它亲自来干，不过随着时间的推移，大而全的 Docker Daemon 进程带来了越来越多的 **问题：**

*   难于变更，对 Docker Daemon 进行维护和升级会对运行中的容器产生影响；
    *   因为所有容器的运行时逻辑都是在 Docker Daemon 中实现的，所以启动和停止 Daemon 会导致宿主机上的所有运行中容器被杀掉。
*   运行的越来越慢。

因此，Docker 公司开始将这个 “大而全” 的 Daemon 进程拆解开，使用 “小而专” 的工具替代它。这一过程十分的符合 UNIX 的设计哲学：小而专的工具可以组装为大型工具！

**Docker 引擎的架构如下：**

[![00-Docker引擎的架构.png](https://github.com/TangBean/MarkdownNotes/raw/master/DockerAndKubernetes/01-Docker%E5%9F%BA%E7%A1%80/pic/00-Docker%E5%BC%95%E6%93%8E%E7%9A%84%E6%9E%B6%E6%9E%84.png)](https://github.com/TangBean/MarkdownNotes/blob/master/DockerAndKubernetes/01-Docker%E5%9F%BA%E7%A1%80/pic/00-Docker%E5%BC%95%E6%93%8E%E7%9A%84%E6%9E%B6%E6%9E%84.png)

**该架构的优势：**

*   对 Docker Daemon 的维护和升级工作不会影响到运行中的容器；
*   在旧的架构中，所有容器的运行逻辑都在 Daemon 中实现，启动和停止 Daemon 都会导致宿主机上所有运行中的容器被杀掉。

**容器与 Daemon 解耦的实现：**

shim 是实现无 Daemon 容器的关键，从上图可知，containerd 通过指挥 runc 来创建新的容器。事实上，每次创建容器时，它都会 fork 一个新的 runc 进程，不过，一旦容器创建完毕，对应的 runc 进程就会退出，否则我们运行多少个容器，就要保持多少个 runc 实例。

可是 runc 进程挂掉了，那我们的容器的父进程是谁呢？此时，containerd-shim 进程登场了，一旦容器进程的父进程 runc 退出，相关联的 containerd-shim 进程就会成为容器的父进程，作为容器的父进程，shim 的部分职责如下：

*   保持所有 stdin 和 stdout 流是开启状态的，这样当 daemon 重启时，容器不会因为 pipe 的关闭而终止；
*   将容器的退出状态反馈给 daemon。


# 镜像（Image）——一个特殊的文件系统
Docker 镜像（Image）就是一个只读的模板。例如：一个镜像可以包含一个完整的操作系统环境，里面仅安装了 Apache 或用户需要的其它应用程序。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。
![image](http://490.github.io/images/20190311_081737.png)
　操作系统分为内核和用户空间。对于 Linux 而言，内核启动后，会挂载 root 文件系统为其提供用户空间支持。而Docker 镜像（Image），就相当于是一个 root 文件系统。

　　Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。 镜像不包含任何动态数据，其内容在构建之后也不会被改变。

　　Docker 设计时，就充分利用 Union FS的技术，将其设计为 分层存储的架构 。 镜像实际是由多层文件系统联合组成。

　　镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。　比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

　　分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

Docker 镜像由多个层组成，每层叠加之后，从外部看起来就是一个独立的对象。镜像内部是一个精简的操作系统，同时还包含应用运行所必须的文件和依赖包。

镜像相当于是一个创建容器的模板，一旦容器从镜像启动以后，二者之间就变成了相互依存的关系，并且在镜像上启动的容器全部停止之前，镜像是无法被删除的。

# 容器（Container)——镜像运行时的实体
镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等 。

　　容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。前面讲过镜像使用的是分层存储，容器也是如此。

　　容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

　　按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据 ，容器存储层要保持无状态化。所有的文件写入操作，都应该使用数据卷（Volume）、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主(或网络存储)发生读写，其性能和稳定性更高。数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此， 使用数据卷后，容器可以随意删除、重新 run ，数据却不会丢失。

# 仓库（Repository）——集中存放镜像文件的地方
镜像构建完成后，可以很容易的在当前宿主上运行，但是， 如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry就是这样的服务。

　　一个 Docker Registry中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。所以说：镜像仓库是Docker用来集中存放镜像文件的地方类似于我们之前常用的代码仓库。

　　通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本 。我们可以通过 <仓库名>:<标签>的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签.。

这里补充一下Docker Registry 公开服务和私有 Docker Registry的概念：

　　Docker Registry 公开服务 是开放给用户使用、允许用户管理镜像的 Registry 服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像。

# Docker 数据卷

容器中的数据管理方式主要有以下两种：

*   数据卷：容器内的数据直接映射到主机环境；
*   数据卷容器：一个有数据卷的容器，用来提供数据卷给其他容器挂载。

## [](https://github.com/TangBean/MarkdownNotes/blob/master/DockerAndKubernetes/01-Docker%E5%9F%BA%E7%A1%80/03-Docker%E6%95%B0%E6%8D%AE%E5%8D%B7.md#%E6%95%B0%E6%8D%AE%E5%8D%B7)数据卷

### [](https://github.com/TangBean/MarkdownNotes/blob/master/DockerAndKubernetes/01-Docker%E5%9F%BA%E7%A1%80/03-Docker%E6%95%B0%E6%8D%AE%E5%8D%B7.md#%E4%BB%80%E4%B9%88%E6%98%AF%E6%95%B0%E6%8D%AE%E5%8D%B7)什么是数据卷

数据卷就是主机中的一个特殊目录，这个目录可以提供给容器使用。这个行为类似于 LInux 的 mount，其实就是将主机中的目录直接映射进容器。

> **mount 的原理**

### [](https://github.com/TangBean/MarkdownNotes/blob/master/DockerAndKubernetes/01-Docker%E5%9F%BA%E7%A1%80/03-Docker%E6%95%B0%E6%8D%AE%E5%8D%B7.md#%E6%95%B0%E6%8D%AE%E5%8D%B7%E7%9A%84%E7%89%B9%E6%80%A7)数据卷的特性

*   数据卷可以在容器之间共享和重用，可以被用来在容器间传递数据；
*   无论是容器内操作还是本地主机的操作，对数据卷内数据的修改会立即生效；
*   对数据卷的更新不会影响到镜像，可以将应用与数据解耦；
*   数据卷会一直存在，即便没有容器使用它，如果一个数据卷没有被容器使用时，我们可以安全的把它删掉。

### [](https://github.com/TangBean/MarkdownNotes/blob/master/DockerAndKubernetes/01-Docker%E5%9F%BA%E7%A1%80/03-Docker%E6%95%B0%E6%8D%AE%E5%8D%B7.md#%E4%BD%BF%E7%94%A8%E6%95%B0%E6%8D%AE%E5%8D%B7)使用数据卷

#### [](https://github.com/TangBean/MarkdownNotes/blob/master/DockerAndKubernetes/01-Docker%E5%9F%BA%E7%A1%80/03-Docker%E6%95%B0%E6%8D%AE%E5%8D%B7.md#%E5%88%9B%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%8D%B7)创建数据卷

```source-shell
docker volume create -d test
```

#### [](https://github.com/TangBean/MarkdownNotes/blob/master/DockerAndKubernetes/01-Docker%E5%9F%BA%E7%A1%80/03-Docker%E6%95%B0%E6%8D%AE%E5%8D%B7.md#%E6%8C%82%E8%BD%BD%E6%95%B0%E6%8D%AE%E5%8D%B7)挂载数据卷

除了使用 `volume` 子命令来管理数据卷以外，我们还可以将本地主机的任意目录挂载到容器内作为数据卷，这种数据卷叫 “绑定数据卷”，此外，还有一种只存在于内存中的 “临时数据卷”。

我们可以在使用 `docker run` 命令时，通过 `--mount` 选项来为容器挂载数据卷，`--mount` 支持 3 中类型的数据卷：

*   `type=volume`：普通数据卷，映射到主机的 `/var/lib/docker/volumes` 路径下；
*   `type=bind`：绑定数据卷，映射到主机的指定目录下；
*   `type=tmpfs`：临时数据卷，只存在于内存中。

此外，我们还可以通过 `-v` 选项来为容器创建数据卷：

> **数据覆盖问题：**
> 
> *   挂载一个空的数据卷到容器的非空目录：容器中目录下的文件会被复制到数据卷中。
> *   挂载一个非空的数据卷到容器的非空目录：容器中目录会显示数据卷中的内容，原来容器中目录下的内容会被隐藏。

# 常用命令

docker ps
列出所有运行中容器。

进入应用内部的系统  4f是编号
docker exec -it 4f bash

docker网络类型：bridge（和主机独立） host none

bridge需要端口映射

docker run -d -p 8888:8080 test/tomcat

-d：表示指定容器后台运行

-p：表示宿主机的8080端口对外映射暴露为8888端口

 docker rm
docker rm [options "o">] <container>  "o">[container...]
docker rm nginx-01 nginx-02 db-01 db-02
sudo docker rm -l /webapp/redis
-f 强行移除该容器，即使其正在运行；
-l 移除容器间的网络连接，而非容器本身；
-v 移除与容器关联的空间。

docker start|stop|restart

docker images
docker images [options "o">] [name]
列出本地所有镜像。其中 [name] 对镜像名称进行关键词查询。


# docker 使用mysql

docker run --name mysql01 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest

（1）进入镜像中的mysql（ti 后面的字符串是mysql镜像ID）

docker exec -ti 2cbb0f246353 /bin/bash

（2）登录mysql

mysql -u root -p

（3）.更改加密方式：_

ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
（4）修改root 可以通过任何客户端连接

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';


# 容器自动退出问题


docker logs -f  容器id 

查看容器日志信息,看最后面发现一个error:
```
ERROR: [1] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least
```


显示max_map_count的值太小了，需要设大到262144

查看max_map_count :
```
cat /proc/sys/vm/max_map_count
65530
 
```

设置max_map_count:
```
sysctl -w vm.max_map_count=262144
vm.max_map_count = 262144
```

重启容器:docker start  容器id或名字


