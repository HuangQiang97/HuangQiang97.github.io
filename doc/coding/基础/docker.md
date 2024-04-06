### 容器

[toc]

##### 原理

* 快速打包技术：标准化（运行时标准、镜像标准）；轻量级（无guest os、共享内核、无真正的容器存在、本质为宿主机上进程、进程调度、内存访问、文件IO都由宿主机完成、docker辅助角色、容器进程都是docker daemon子进程，无虚拟化、无完整OS、共享bootfs、无需引导与加载os、快，本质为进程无虚拟化损耗、namespace隔离无需os损耗、rootfs(发行版)存储空间小、）；易移植（标准化、rootfs+app+config+lib、完整依赖，沙河、集装箱（隔离、搬运）、易于拓展、功能单一、解耦易维护，paas，一次打包到处运行、多容器高性能，虚拟机可以占据全部性能）
* 架构：client(命令)、docke_host(daemon进程、images、container)、远程仓库
* Linux Container：内核虚拟化技术，轻量级的虚拟化，隔离进程和资源
* namespce：约束进程的动态表现来创建边境，PID Namespace（init pid=1、层级结构（同级、上层下层）、clone进程时添加clone_newPID参数、可见性、发送信号）、Mount Namespace （隔离看到的挂载点视图、视图在挂在后才改变、chroot、容器镜像、rootfs、发行版、执行环境文件系统（bin、lib、root、无内核）、一致性）、Network namespace 让进程只看到当前namespace中的网络设备、隔离路由表。不存在真实的容器、安全、多容器高性能。
* cgroups：容器进程与主机上其它进程平等竞争、可能占据主机全部资源、多子系统、一组进程cpu、memory资源限制、接口为文件系统、树结构（子系统、容器）、参数（docker run时指定）。V1(子系统协调困难)。

##### 运行

* 镜像：文件系统、层:发行版（只读、whileout）、共享、unionfs（分层、修改为一层、增量rootfs、每一层都挂载在宿主机相应的目录下、层次覆盖、继承）、rootfs+app+config+lib、完整依赖,

* 容器：init层（ro、wh、host信息（host属于只读镜像但用户常修改此值，并不希望提交，commit只提交可读可写层））+readwrite（数据修改发生在此，增、改变（copy on write）、删除（wo文件、遮挡只读层文件）、commit保存可读可写层而只读层不变,将读写层+镜像层打包为镜像，其中镜像层共享不占用空间），写时复制、限制资源及试图的进程。

* 优势：bootfs,rootfs,无guest os,

* Client-Server结构的系统，Docker的守护进程。通过Socket从客户端访问！Docker-Server接收到Docker-Client的指令，就会执行这个命令

  <img src="D:/sync/Note/%25E9%259D%25A2%25E7%25BB%258F.assets/docker-stages.png" alt="docker-image" style="zoom:50%;" />

* docker exec 开启一个新的终端，新的进程属于容器对应的namespace和cgroups但是其父进程是docker daemon而不是pin=1，一个进程的 Namespace 信息在宿主机上以一个文件的方式存在，可以选择一个进程并加入他说在的namespace，

* attach 进入正在执行的终端。

* pin=1进程收到结束信号，直接给其子进程发送kill信号后容器结束。

##### 命令行

```shell
docker system prune -f # 删除停止容器和网络和镜像
docker image prune -a # 删除未被使用镜像
docker volume prune -f # 删除空闲volune
docker container run --rm  -it image_name args # 创建的容器在使用结束后自动删除容器
```

`CMD `与`ENTRYPOINT`联合使用，`ENTRYPOINT` 设置执行的命令，`CMD`传递参数：

```shell
ENTRYPOINT ["echo"]
CMD []

docker container run -it -rm image_name hello # 输出hello
```

#####  信号处理

pin=1：one process per comntainer，只有pid=1可控，init进程、守护进程、祖先、第一个用户态进程、管理、生命周期、信号、docker stop和kill（默认）发送sigterm（正常终止信号缺省行为退出，自定义handler资源清理、等待、sigkill,传播给子进程、优雅），kill -9强制，忽略（sigstop，sigkill除外）+捕获（自定义、sigstop，sigkill除外）+缺省，处理孤儿和回收僵尸进程(dumb-init三方)。

Kill：sig_ignore(内部调用&默认handler&目标为pin=0)则忽略信号，kill -9 1再容器内部不工作。kill 1：如果pin=1实现了堆sigterm的自定义则响应否则不响应。

正常停止容器：pin=1收到sigtrem(缺省行为是释放资源后退出)，其他进程收到pin=1发出的sigkill强制结束。自定义pin=1的singterm的hander，向其他进程转发sigterm而非sigkill,资源清理，graceful shutdown。

shell格式：命令行格式、先启动shell再执行服务程序，pin=1线程为shell，sigterm,handler，未提供、等待，sigkill强制退出。exec格式：字符串集合、pin=0线程为目标程序，docker run命令行cmd使用exec格式。

僵尸进程：zombie->exit，结束、资源已释放，占据PID,等待父进程(直接父类)回收(父进程收到子进程结束信号、僵尸不想赢kill信号、wait()、waitpid)、父进程获取子进程退出信息（nginx）、资源泄露(PID)。父进程结束：孤儿进程、收养、init进程。

默认情形下，docker提供一个隐含的entrypoint：“/bin/sh -c”，如果不指定entrypoint，cmd内容就是entrypoint的参数：/bin/sh -c python app.py。Docker 容器的启动进程为 ENTRYPOINT，而不是 CMD。

##### 文件

* 文件复制时如果是复制到文件夹中需要指明`/src/`，如果只写`/src`会将`src`当作文件，将文件复制到镜像中并重命名为`src`。
* 可写层是和特定的容器绑定，Data Volume由Docker管理（挂载方便、共享、读写权限管理），Bind Mount用户指定存储位置，`dockerfile`中的``VOLUME ["app"]`设置项是将主机目录挂载到容器中的目录app上，本机会自动创建一个随机名字的volume，去存储我们在Dockerfile定义的volume，默认在`/var/lib/docker/volumes`下；命令行通过`-v`可以可以手动的指定本机目录，以及对应于容器内的路径，这个路径是可以任意的，不必需要在Dockerfile里通过VOLUME定义。
* Data Volume挂载原理：当容器进程被创建之后，尽管开启了 Mount Namespace，但是在它执行 chroot之前，容器进程一直可以看到宿主机上的整个文件系统。只需要在 rootfs 准备好之后，在执行 chroot 之前，把 Volume 指定的宿主机目录挂载到指定的容器目录在上，这个 Volume 的挂载工作就完成了。Mount Namespace 已经开启了。这个挂载事件只在这个容器里可见。在宿主机上是看不见容器内部的这个挂载点的，这就保证了容器的隔离性不会被 Volume 打破，同时commit也不会提交挂载目录。挂载的底层实现为Bind Mount将一个目录或者文件，而不是整个设备，挂载到一个指定的目录上（文件修改发生在宿主机的对应目，不会影响容器镜像的内容。

##### 构建

* dockerfile：基础镜像、环境(run安装软件、变量arg、环境变量env、多行合并）、交互（网络、端口、挂载）、加入目标程序文件（add,copy）、启动(cmd命令行覆盖，entrypoint、联合使用)。等同于 Docker 使用基础镜像启动了一个容器，然后在容器中依次执行 Dockerfile 中的原语。

* 缓存：当dockerfile中某一层发生改变，该层以及其之后的所有层都会重新构建，不使用缓存，即使之后层并未发生变化。可以将不经常的层放到前面，将经常变化的层放到后面，合理利用缓存，加快构建速度。
* 多阶段构建：适用于程序的编译操作复杂，需要大量辅助程序，而编译后的程序并不需要这些辅助程序，通过多阶段构建，将编译阶段从最终的结果中分离，将编译结构构建进最终的输出镜像，精简结果镜像。
* 用户：容器默认root权限，/root映射到容器，/etc/sudoers映射到容器；run创建用户组和用户，更改目标文件归属，切换用户。

##### 网络

* 容器访问外网：veth-pair虚拟设备接口，成对出现的，一端连接一个namespace(veth_container放入容器network namespace,veth-host放入host network namespace，实现host通信)；容器数据在网桥处进行网络地址转换(bridge+NAT,将veth-host加入docker0,容器+docker0构成子网)，借助本地主机地址访问外网（容器-(veth-pair)->docker0-(nat)->eth0）。NAT用于解决IP地址不足的问题，给一系列设备分配私有IP，当他们想要访问外网时，利用NAT将私有发送地址转换为共有地址，同样当受到外界发送回来的数据是将目的IP转换为私有IP，相当于一些列设备公用一个IP地址。

* 容器默认连接到名称为bridge的bridge桥接网络(veth)，bridge网络还连接本地主机端（eth）。一个容器可以连接到多个bridge上，拥有多个私有IP地址。

  自定义的bridge可以实现DNS的功能（iptable），可以直接使用与该bridge连接的容器名直接通信，而默认bridge(docker0)未实现该功能。

  ```
  docker network connect bridge_name container_name #连接
  docker network dis connect bridge_name container_name #断开连接
  ```

* Network namespace隔离,IP地址，路由表。

* host网路使用和主机一样的网络，类似于容器、主机使用相同网络设备，免去转换，提升性能（注意多个容器监听相同端口导致冲突）。none网络下的容器无法实现使用网络（内部、外部网络都无法使用，常用于容器编排）。

##### 端口

* 利用IPtable实现主机端口到容器私有地址及其端口的映射(8080 to:172.17.0.2:80)。`EXPOSE`并不是真正的暴露的端口，只是为了告诉使用者该容器应该暴漏该端口，在启动容器时要使用`-p`暴漏端口。

##### compose

* docker -compose服务更新文件发生改变后使用`docker-compose up `重新启动服务，会自动更新发生修改的容器，未经过修改的容器不会被重启。如果某个容器对应的镜像发生改变需要手指指定`build`参数：`docker-compose up -d --build`重新构建镜像并重启容器。如果是删除镜像需要添加参数以删除多余的容器：`docker-compose up -d --remove orphans`。当需要重启时使用`docker-compose restart`重启所有服务。

* docker-compose默认会为文件中定义的所有服务定义一个bridge，所以容器将可以使用服务名ping通，但是不同docker-compose间无法ping通（可以ping通本地主机，bridge默认会将本地网络自动加入）。

* 水平扩展：`docker-compose up -d --scale flask=3`将名为`flask`的服务扩展为3份，当通过服务名（DNS）连接flask时会将连接请求均分到三个服务，实现简单的发在均衡。

* 隐藏关键信息：对于不希望出现在docker-compose中的敏感信息，可以使用`${arg_name}`替换，再在相同文件夹下创建`.env`文件，写入`arg_name=abc`，在启动compose时会自动替换，可以使用`docker-compose config`检查替换后的compose文件。

* 服务依赖： 添加配置项`depends_on`指定该服务依赖另一个服务，需要在依赖服务启动后再启动该服务。但是服务启动不代表能正常提供服务，可以设置健康检查，`HEALTHCHECK`配置项，更细化检查指标。

##### 安全

* capbilities:原始两类（root,非root）；capability权限细分（一个特权操作对应一个capability），主机上：（root用户进程默认包含全部cap，非root进程默认无任何cap），容器：（privileged容器可以进行全部特权操作，容器中root用户进程默认只开启部分cap、安全、按需设置cap）
* user namespace:容器中root用户进程默认只开启部分cap更改修改主机上关键文件，主机root用户与容器root用户uid相同，共享内核导致软件漏洞危害主机系统；指定普通uid（使用宿主机上该id对应的用户、主机上uid共享，多容器冲突、资源受限）；usernamsespce(隔离uid、gid，容器-主机映射、容器中uid=0(再容器内拥有一定特权)->宿主机上普通用户（即使逃逸权限有限），指定映射范围防止冲突)；以非root创建管理容器（docker以非root执行）

##### 容器编排

* 一组联合挂载在 /var/lib/docker/aufs/mnt 上的 rootfs（容器镜像、静态视图、开发者关注重点）+ 一个由 Namespace+Cgroups 构成的隔离环境，（容器运行时，动态视图）。

* Master（负责api服务、调度、编排；如何编排、管理、调度用户提交的作业）+Node结构；kubelet 通过CRI（Container Runtime Interface）的远程调用接口负责同容器运行时打交道，gri接口定义了容器运行时的各项核心操作(通用性，不局限于docker)，通过 OCI 这个容器运行时规范同底层的Linux 操作系统进行交互；gRPC 协议同一个叫作 Device Plugin 的插件进行交互，是 Kubernetes 项目用来管理 GPU 等宿主机物理设备的主要组件；通过CNI（Container Networking Interface）和CSI（Container Storage Interface）实现对网络和存储的管理。

  <img src="D:/sync/Note/%25E9%259D%25A2%25E7%25BB%258F.assets/image-20220310221912531.png" alt="image-20220310221912531" style="zoom:50%;" />



* 在大规模集群中的各种任务之间，实际上存在着各种各样的关系;虚拟机（粗粒度）->容器(细粒度);以统一的方式来定义任务之间的各种关系;将非常频繁的交互和访问的容器划为一个pod,Pod 里的容器共享同一个 Network Namespace、同一组数据卷，从而达到高效率交换信息的目的;Service服务(Web应用与数据库、故意不在同一机上部署、容灾)；实现多个pod间的交互给 Pod 绑定一个 Service 服务，而 Service 服务声明的 IP 地址等信息是“终生不变”的。Service 服务的主要作用，就是作为 Pod 的代理入口，从而代替 Pod 对外暴露一个固定的网络地址。外部应用只关心service,Service Kubernetes 负责后端真正代理的 Pod 的 IP 地址、端口等信息的自动更新、维护以及负载均衡；Kubernetes 项目定义容器间关系和形态的主要方法。
* 通过一个“编排对象”，比如 Pod、等，来描述你试图管理的应用；定义一些“服务对象”，比如 Service等负责具体的平台级功能。调度（运行起来）->编排（自动处理容器间的关系）