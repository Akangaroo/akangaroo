[TOC]

## Docker简介

### 是什么

一款产品从开发到上线，从操作系统，到运行环境，再到应用配置。作为开发+运维之间的协作我们需要关心很多东西，这也是很多互联网公司都不得不面对的问题，特别是各种版本的迭代之后，不同版本环境的兼容，对运维人员都是考验Docker之所以发展如此迅速，也是因为它对此给出了一个标准化的解决方案。环境配置如此麻烦，换一台机器，就要重来一次，费力费时。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。开发人员利用Docker可以消除协作编时“在我的机器，上可正常工作”的问题。

传统上认为，软件编码开发/测试结束后，所产出的成果即是程序或是能够编译执行的二进制字节码等(java为例)。而为了让这些程序可以顺利执行，开发团队也得准备完整的部署文件，让运维团队得以部署应用程式，开发需要清楚的告诉运维部署团队，用的全部配置文件+所有软件环境。不过，即便如此，仍然常常发生部署失败的状况。**Docker镜像的设计，使得Docker得以打破过去「程序即应用」的观念。透过镜像(images)将作业系统核心除外，运作应用程式所需要的系统环境，由下而上打包，达到应用程式跨平台间的无缝接轨运作**。

==简而言之就是：解决了**运行环境**和**配置问题**软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。==

### 能干嘛

#### 虚拟机技术

虚拟机(virtual machine)就是带环境安装的一种解决方 案。它可以在一种操作系统里面运行另一种操作系统，比如在Windows系统里面运行Linux系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。

![image-20210120161028418](http://img.akangaroo.cn/docker/image-20210120161028418.png)

虚拟机的缺点：1. 资源占用多；2. 冗余步骤多；3. 启动慢

#### 容器虚拟化

由于前面虚拟机存在这些缺点，Linux发展出了另一种虚拟化技术: Linux容器(Linux Containers， 缩写为LXC)Linux容器不是模拟一一个完整的操作系统，而是对进程进行隔离。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。

![image-20210120161217867](http://img.akangaroo.cn/docker/image-20210120161217867.png)

#### 上述两者区别

- **传统虚拟机技术**是虚拟出一套硬件后，在其上**运行一个完整操作系统**，在该系统上**再运行所需应用进程**;
- 而**容器**内的**应用进程直接运行于宿主的内核**，容器内==没有自己的内核==，而且也==没有进行硬件虚拟==。因此容器要比传统虚拟机更为轻便。
- 每个容器之间**互相隔离**，每个容器有自己的文件系统，容器之间进程不会相互影响，能区分计算资源。



### 基本组成

镜像（image）：Docker镜像(Image) 就是一个**只读**的模板。镜像可以**用来创建Docker容器**，**一个镜像可以创建很多容器**。

容器（container）：Docker利用容器（Container） **独立运行的一个或一组应用**。容器是用镜像创建的运行实例。

仓库（repository）：集中存放镜像的地方。



## Docker安装

### 安装步骤

docker安装前提：centos7的系统为64位，系统内核版本位3.10以上，可通过`uanme -r`命令查看。

[centos7官方文档安装](https://docs.docker.com/engine/install/centos/)

1. 如果之前安装过，卸载旧版本。

   ```shell
   	sudo yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   ```

2. 安装`yum-utils`软件包（提供`yum-config-manager` 应用程序）并设置**稳定的**存储库。

   ```shell
   sudo yum install -y yum-utils
   
   sudo yum-config-manager \
       --add-repo \
       https://download.docker.com/linux/centos/docker-ce.repo
   ```

3. 安装最新版docker引擎

   ```shell
   sudo yum install docker-ce docker-ce-cli containerd.io
   ```

4. 启动docker

   ```shell
   sudo systemctl start docker
   ```

5. 查看docker版本

   ```shell
   docker version
   ```

   

### 配置阿里云镜像

1. 登录阿里云

2. 搜索【镜像与容器】，进入后点击【容器镜像服务】

   ![image-20210118111833657](http://img.akangaroo.cn/docker/image-20210118111833657.png)



3. 进入后，点击镜像加速器，按照右边的最下面文档，复制粘贴，即可生效。

   ![image-20210118111954940](http://img.akangaroo.cn/docker/image-20210118111954940.png)

使用`docker run hello-world`命令。



## 运行底层原理

### 如何工作

Docker是一个Client- Server结构的系统，Docker守护进程运行在主机上，然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。容器，是一个运行时环境。

![image-20210120162055792](http://img.akangaroo.cn/docker/image-20210120162055792.png)

### 为什么Docker比虚拟机运行快

1. docker有着比虚拟机更少的抽象层。由于**docker不需要Hypervisor实现硬件资源虚拟化**，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。

2. docker利用的是宿主机的内核，而不需要Guest OS。因此，当新建一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核。仍而避免引寻、加载操作系统内核返个比较费时费资源的过程，当新建一个虚拟机时，虚拟机软件需要加载GuestOS，这个新建过程是分钟级别的。而**docker由于直接利用宿主机的操作系统**，则省略了这个过程,因此新建一个docker容器只需要几秒钟。

   ![image-20210120162506375](http://img.akangaroo.cn/docker/image-20210120162506375.png)

   ![image-20210120162605357](http://img.akangaroo.cn/docker/image-20210120162605357.png)

## Docker常用命令

### 帮助命令

```shell
docker version

docker info

docker --help
# 举例
docker --help images
```



### 镜像命令

```shell
# 列出主机上的镜像
docker images [OPTIONS] 镜像名 
```

常用参数说明：

- `-a`：列出本地所有镜像（包含中间映像层）
- `-q`：只显示镜像ID
- `--digests`：显示镜像的摘要信息
- `--no-trunc`：显示完整的镜像信息



```shell
# 搜索镜像
docker search [OPTIONS] 镜像名
```

常用参数说明：

- `–no-trunc`：显示完整的镜像描述

- `-f=stars=XX` ：列出stars数不小于XX的镜像

- `-f=is-official=true` ：列出官网的镜像

- `-f=is-automated=true` ：列出automated build类型的镜像

  

```shell
# 拉取镜像 :TAG为版本，不写的话默认最新
docker pull 镜像名[:TAG]
```



```shell
# 删除镜像
docker rmi 镜像名（镜像ID）
```

`-f`代表强制删除

- `docker rmi -f 镜像名（或ID）`：删除单个
- `docker rmi -f 镜像名:TAG1 镜像名:TAG2` ：删除多个
- `docker rmi -f $(docker images -qa)` ：删除全部



### 容器命令

==有镜像才能创建容器，这是根本前提==

```shell
# 创建并启动容器
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

常用参数说明

- `--name "名字"`：为容器指定名字
- `-d`：后台运行容器
- ==-i==：以交互模式运行容器，通常与-t一起使用
- ==-t==：为容器重新分配一个伪输入终端，通常与-i一起使用
- `-P(大写)`：随机端口映射
- ==-p==（小写）：指定端口映射
  - ip:hostPort:containerPort
  - ip::containerPort
  - ==hostPort:containerPort==
  - containerPort

> 如下图所示，-it进入了centos的内部，并为他命名mycentos

![image-20210121142842331](http://img.akangaroo.cn/docker/image-20210121142842331.png)

**此处我用的退出方式是exit，有两种退出容器的方式。**

- `exit`：容器停止退出
- `ctrl+p+q`：容器不停止退出

所以使用docker ps的时候不会出现，`docker ps`命名解释如下

-----------------------

```shell
# 列出当前所有运行的容器
docker ps [OPTIONS]
```

常用参数说明：

- `-a`：列出当前所有正在运行的容器+历史上运行过的容器
- `-l`：显示最近创建的容器
- `-n`：显示最近n个创建容器
  - docker ps -n 2
- `-q`：只显示容器Id
- `--no-trunc`：不截断输出

---------

**容器退出之后改如何启动？**

```shell
docker start ID或名字
```

![image-20210121143414233](http://img.akangaroo.cn/docker/image-20210121143414233.png)

--------------

**启动后发现没有进入centos的内部，如何进入容器内部？**

```shell
docker exec -it 容器ID /bin/bash

docker attach 容器ID
```

两行命令都可以进入容器，但exec也和attach有一定的区别：

`docker exec -it 容器ID bashShell` 直接让容器运行命令，不进入容器

`docker exec -it 容器ID /bin/bash` 可进入容器

`docker attach 容器ID` 重新进入容器

官网解释：

> Run a command in a running container：翻译过来，在一个正在运行的容器中执行命令，exec是针对已运行的容器实例进行操作，在已运行的容器中执行命令，不创建和启动新的容器，退出shell不会导致容器停止运行。
>
> Attach local standard input, output, and error streams to a running container：翻译过来，将本机的标准输入（键盘）、标准输出（屏幕）、错误输出（屏幕）附加到一个运行的容器，也就是说本机的输入直接输到容器中，容器的输出会直接显示在本机的屏幕上，如果退出容器的shell，容器会停止运行。

可通过如下的案例理解：



![image-20210121150446953](http://img.akangaroo.cn/docker/image-20210121150446953.png)



```shell
# 重启容器
docker restart ID或容器名

# 停止容器
docker stop ID或容器名
docker stop $(docker ps -a -q) # 一次性停止所有容器

# 强制停止容器
docker kill ID或容器名

# 删除已经停止的容器
docker rm ID
docker rm -f $(docker ps -a -q) # 一次性删除全部
docker ps -a -q | xargs docker rm # 一次性删除全部
```



```shell
# 查看容器日志
docker logs -f -t --tail 3 ID或容器名

# 参数说明
-f 不停的追加
-t 显示时间
--tail X 显示最后几条日志
```



```shell
# 查看容器内运行的进程
docker top ID或容器名
```



```shell
# 查看容器内部细节
docker inspect ID或容器名
```



```shell
# 拷贝
docker cp 容器ID:容器内部路径 目的主机路径
```

![image-20210121152823332](http://img.akangaroo.cn/docker/image-20210121152823332.png)





## Docker镜像

### UnionFS

UnionFS (联合文件系统) : Union文件系统(UnionFS) 是一种分层、 轻量级并且高性能的文件系统，它**支持对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像(没有父镜像)，可以制作各种具体的应用镜像。

特点：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统.叠加起来，这样最终的文件系统会包含所有底层的文件和目录

### Docker镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的， 包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system)，在bootfs之 上。包含的就是典型Linux系统中的/dev, /proc, /bin, /etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu, Centos等。

![image-20210121155024524](http://img.akangaroo.cn/docker/image-20210121155024524.png)

对于一个精简的OS, rootfs可以很小， 只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel， 自己只需要提供rootfs就行了。由此可见对于不同的linux发行版, bootfs基本是一 致的,rootfs会有差别,因此不同的发行版可以公用bootfs。



### 为什么分层

最大的一个好处就是**共享资源**

比如:有多个镜像都从相同的base镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像,同时内存中也只需加载一份base镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。



### 特点

Docker镜像都是只读的，容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作"容器层”，“容器层”之下的都叫“镜像层”。



### commit操作

docker commit提交容器副本使之成为一个新的镜像

```shell
docker commit -m "提交的描述信息" -a "作者" 容器ID 要创建的目标镜像名:[标签名]
```

案例：

1. `docker run -it -p 8080:8080 --name mytomcat tomcat` 运行tomcat。
2. 输入IP:8080查看tomcat信息
3. 如果出现404错误，ctrl+P+Q不停止退出tomcat，然后输入`docker exec -it tomcat容器的ID /bin/bash`进入tomcat。
   1. `ls -l`查看，发现有webapps和webapps.dist，发现webapps的内容在webapps.dist目录下
   2. `rm -rf webapps`删除webapps目录
   3. `mv webapps.dist webapps`指定将webapps.list重命名为webapps目录
   4. `docker start mytomcat`重启tomcat
   5. 输入IP:8080查看

ps：开放8080端口，阿里云安全组配置

以上成功完成后。

1. 输入`docker exec -it mytomcat /bin/bash`进入容器内
2. 接着`cd webapps`进入文件夹下
3. `rm -rf docs`删除docs文件夹。
4. 清除浏览器的缓存，刷新页面，点击Documentation发现404。

![image-20210121162721643](http://img.akangaroo.cn/docker/image-20210121162721643.png)

接着进行commit操作，如下图所示，通过`docker images`可以发现成功打打包。

![image-20210121163031177](http://img.akangaroo.cn/docker/image-20210121163031177.png)



## Docker容器数据卷

### 作用

- 持久化数据
- 容器间继承+共享数据

### 数据卷

#### 命令形式

```shell
docker run -it -v /宿主机绝对路径:/容器内路径 ID或镜像名
# 如果没有此目录，此命令会新建目录，如果宿主机上有此目录，文件会和容器共同使用，而如果容器有此目录，会删掉原来的目录然后再新建，原本里面的内容会被清空，而宿主机不会

docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名 
# 带权限的，readonly，宿主机可读写，容器可读不可写。
```

案例：

1. `docker run -it -v /tmp/myDataVolume:/tmp/containerVolume --name mycentos2 centos`
2. `cd /tmp/containerVolume`目录下
3. `vi aaa.txt`
4. 上述操作完成后到宿主机`cd /tmp/myDataVolume`目录下，查看确实经过修改。
5. 另外，还可以通过`docker inspect ID或容器名`查看

> 容器退出后，主机修改数据后也能进行同步
>
> 类似Redis中的ROF和ADB文件

![image-20210121172243537](http://img.akangaroo.cn/docker/image-20210121172243537.png)

通过`docker inspect ID或容器名`查看的结果如下

![image-20210121172929884](http://img.akangaroo.cn/docker/image-20210121172929884.png)

#### Dockerfile形式

1. 在/tmp目录下`vim Dockerfile`新建Dockerfile文件

2. 里面的内容为

   ```dockerfile
   FROM centos
   VOLUME ["dataVolumeContainer1","dataVolumeContainer2"]
   CMD /bin/bash
   ```

3. 通过build生成镜像`docker build -f /tmp/mydocker/Dockerfile -t akangaroo/centos .`==（最后的.表示当前目录下，如果此时所在目录的Dockerfile文件名为Dockerfile，-f /tmp/mydocker/Dockerfile 可省略不写）==

4. 进入容器，发现存在dataVolumeContainer1和dataVolumeContainer2目录

   ![image-20210121173812661](http://img.akangaroo.cn/docker/image-20210121173812661.png)

5. `docker inspect 容器ID`查看对应的宿主机目录

   ![image-20210121173949104](http://img.akangaroo.cn/docker/image-20210121173949104.png)

6. 进入目录，查看是否成功，如下图所示。

   ![image-20210121174245041](http://img.akangaroo.cn/docker/image-20210121174245041.png)

### 容器内数据卷

作用：容器间传递共享（**--volumes-from**参数）

案例：

1. 以上一步新建的akangaroo/centos为模板运行容器dc01/dc02/dc03

2. `docker run -it --name dc01 akangaroo/centos`运行dc01，在dataVolumeContainer2下编写dc01.txt文件

3. 再创建dc02和dc03继承dc01`docker run -it --name dc02 --volumes-from dc01 akangaroo/centos`和`docker run -it --name dc03 --volumes-from dc01 akangaroo/centos`

4. 选择在dc03中更改dc01.txt的信息，然后新增文件dc03.txt。

5. 运行dc02后发现都生效了。

   

- 除此之外，删除dc01后依然可以传递数据
- 删除dc02依然可以传递数据
- 新建dc04继承dc03后再删除dc03依然可以传递数据

> 结论：容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止

![image-20210121180507882](http://img.akangaroo.cn/docker/image-20210121180507882.png)





## Dockefile

### 是什么

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

构建的步骤：

1. 编写Dockerfile文件
2. docker build
3. docker run

文件的格式：

[centos](https://hub.docker.com/_/centos)点击进入后，可以看见下图所示的链接。

![image-20210121181443037](http://img.akangaroo.cn/docker/image-20210121181443037.png)

```dockerfile
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20201113" \
    org.opencontainers.image.title="CentOS Base Image" \
    org.opencontainers.image.vendor="CentOS" \
    org.opencontainers.image.licenses="GPL-2.0-only" \
    org.opencontainers.image.created="2020-11-13 00:00:00+00:00"

CMD ["/bin/bash"]
```



### Dockerfile构建过程解析

#### 基础知识

1. 每条保留字指令都必须为**大写字母且后面要跟随至少一个参数**
2. 指令按照从上到下，顺序执行
3. \#表示注释
4. **每条指令都会创建一个新的镜像层，并对镜像进行提交**

#### Docker执行Dockerfile流程

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似`docker commit`的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

#### 小总结

 从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段,
* Dockerfile是软件的原材料
* Docker镜像是软件的交付品
* Docker容器则可以认为是软件的运行态。

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

![image-20210121194757448](http://img.akangaroo.cn/docker/image-20210121194757448.png)

1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;
2. Docker镜像，在用Dockerfile定 义-一个文件之后，docker build时会产生- - 个Docker镜像，当运行Docker镜像时，会真正开始提供服务;
3. Docker容器，容器是直接提供服务的。



### 常用保留字指令

- FROM：基础镜像，当前镜像是基于哪个镜像的
  - scratch（base镜像）：DockerHub中99%的镜像都是通过在base镜像中安装和配置需要的软件构建出来的。
- MAINTAINER：镜像维护者的姓名和邮箱
- RUN：容器构建时需要运行的命令
- EXPOSE：当前容器对外暴露出的端口
- WORKDIR：指定在容器创建后，终端默认登录的进来的工作目录
- ENV：用来构建镜像构建过程中设置环境变量
- ADD：将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
- COPY：类似ADD，拷贝文件和目录到镜像中，但不会解压。将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置
- VOLUME：容器数据卷，用于数据保存和持久化工作
- CMD：指定一个容器运行时要运行的命令，Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被`docker run`之后的参数替换.
- ENTRYPOINT：指定一个容器启动时要运行的命令
- ONBUILD：当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发



### 自定义centos

1. 原生的centos下面没有vim和ifconfig相关的命令

   ![image-20210121202608473](http://img.akangaroo.cn/docker/image-20210121202608473.png)

2. 在/tmp/mydocker下编写Dockerfile

   ```dockerfile
   FROM centos
   MAINTAINER akangaroo<1048831947@qq.com>
   
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   
   RUN yum -y install vim
   RUN yum -y install net-tools
   
   EXPOSE 80
   
   CMD echo $MYPATH
   CMD echo "success------ok"
   CMD /bin/bash
   ```

3. 构建`docker build -t mycentos2:1.0 .`，如下图，可发现，每执行一次命令就生成一个镜像，虽然CMD都执行了，但只有最后一个生效。

   ![image-20210121202929249](http://img.akangaroo.cn/docker/image-20210121202929249.png)

4. `docker run -it mycentos:1.0`运行镜像，可以发现都装配成功了。

   ![image-20210121203250776](http://img.akangaroo.cn/docker/image-20210121203250776.png)

5. `docker history 镜像ID`列出镜像的变更历史

   ![image-20210121203528429](http://img.akangaroo.cn/docker/image-20210121203528429.png)



### 自定义tomcat

1. `mkdir /tmp/mytomcat`创建该文件夹

2. 在该目录下新建`a.txt`，将tomcat和jdk放入改文件夹

3. 在该目录下编写Dockerfile

   ```dockerfile
   FROM centos
   MAINTAINER akangaroo<1048831947@qq.com>
   
   #把宿主机当前上下文的c.txt拷贝到容器/usr/local路径下
   COPY a.txt /usr/local/cincontainer.txt
   
   ADD apache-tomcat-9.0.41.tar.gz /usr/local
   ADD jdk-8u281-linux-x64.tar.gz /usr/local
   
   #安装vim编辑器
   RUN yum -y install vim
   
   #设置工作访问时候的WORKDIR路径,登录落脚点
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   
   #配置java与tomcat环境变量
   ENV JAVA_HOME /usr/local/jdk1.8.0_281
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.41
   ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.41
   ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
   
   #容器运行时监听的端口
   EXPOSE 8080
   
   #启动时运行 tomcat
   #第一种写法
   # ENTRYPOINT ["/usr/local/apache-tomcat-9.0.41/bin/startup.sh"]
   #第二种写法
   # CMD ["/usr/local/apache-tomcat-9.0.41/bin/catalina. sh","run"]
   #第三种写法
   CMD /usr/local/apache-tomcat-9.0.41/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.41/bin/logs/catalina.out
   ```

4. `docker build -t akangaroo/tomcat .`

   ![image-20210121211914257](http://img.akangaroo.cn/docker/image-20210121211914257.png)

5. 通过如下命令运行

   ```shell
   docker run -d -p 8080:8080 --name mytomcat9 -v /tmp/mytomcat/test:/usr/local/apache-tomcat-9.0.41/webapps/test \
   > -v /tmp/mytomcat/logs:/usr/local/apache-tomcat-9.0.41/logs \
   > --privileged=true akangaroo/tomcat
   
   # --privileged=true赋予扩展权限
   ```

6. 进入/tmp/mytomcat/test目录，创建WEB-INF并且新建web.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                         http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
     version="4.0"
     metadata-complete="true">
   
     <display-name>Tomcat akangaroo</display-name>
       
   </web-app>
   
   ```

7. 返回test目录，新建a.jsp

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" language="java" %>
   <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
   <html>
   <head>
       <meta http-equiv="Content-Type" content="text/html" charset="UTF-8">
       <title>Insert title here</title>
   </head>
   <body>
       -----------welcome-----------
       <%="i am in docker tomcat self"%>
       <br>
       <br>
       <%System.out.println("=========docker tomcat self");%>
   </body>
   </html>
   ```

8. 访问后结果如下

   ![image-20210121221331346](http://img.akangaroo.cn/docker/image-20210121221331346.png)

9. 进入宿主机的/tmp/mytomcat/test/log文件夹，查看`cat catalina.out`日志

   ![image-20210121221538507](http://img.akangaroo.cn/docker/image-20210121221538507.png)

   

## 常用软件安装

### mysql

```shell
docker pull mysql:5.7

docker run -p 3306:3306 --name mysql \
> -v /opt/mysql/conf:/etc/mysql/conf.d \
> -v /opt/mysql/logs:/logs \
> -v /opt/mysql/data:/var/lib/mysql \
> -e MYSQL_ROOT_PASSWORD=123456 \
> -d mysql:5.7

```

成功后可以通过`docker exec -it mysql /bin/bash`进入容器内容

输入`mysql -uroot -p123456`即可成功登录

### redis

将redis.conf文件拷贝到/opt/myredis/conf文件夹下，并将其中的`bind 127.0.0.1`注释

![image-20210122001311692](http://img.akangaroo.cn/docker/image-20210122001311692.png)

```shell
docker pull redis

docker run -p 6379:6379 --name redis -v /opt/myredis/data:/data -v /opt/myredis/conf:/usr/local/redis/conf -d redis redis-server /usr/local/redis/conf/redis.conf --appendonly yes

# 通过如下命令进入终端
docker exec -it redis redis-cli
```

输入`set k1 v1`，接着`shutdown`退出

在宿主机的目录`/opt/myredis/data`文件夹下查看，发现已经持久化。

![image-20210122001455597](http://img.akangaroo.cn/docker/image-20210122001455597.png)

### nginx

```shell
docker pull nginx

# 运行镜像
docker run --name nginx-test -p 80:80 -d nginx

# 在宿主机新建目录
mkdir -p /opt/nginx/html /opt/nginx/logs /opt/nginx/conf

# 将下面两个复制到/opt/nginx/conf文件夹下
cp 容器ID:/etc/nginx/nginx.conf /opt/nginx/conf
cp 容器ID:/etc/nginx/conf.d /opt/nginx/conf

# 运行nginx
docker run -d -p 80:80 --name nginx -v /opt/nginx/conf/conf.d/:/etc/nginx/conf.d -v /opt/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /opt/nginx/logs/:/var/log/nginx -v /opt/nginx/html:/usr/share/nginx/html nginx

```

在`/opt/nginx/html`目录下新建`index.html`文件

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Docker搭建Nginx</title>
</head>
<body>
    <h1>人生得意须尽欢</h1>
</body>
</html>
```

输入ip访问

![image-20210122005618782](http://img.akangaroo.cn/docker/image-20210122005618782.png)

