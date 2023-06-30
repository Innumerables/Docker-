#### Docker镜像（NONE)

```
Docker 镜像中的有效none镜像 和无效none镜像
有效的 none 镜像：当一个镜像被其他镜像所依赖时，即使它没有标签（tag），它仍然被视为有效的 none 镜像。这通常发生在多阶段构建中，其中一个构建阶段生成的镜像在后续的构建阶段中被使用。
无效的 none 镜像：当一个镜像既没有标签（tag），也没有其他镜像依赖它时，它被视为无效的 none 镜像。这种镜像通常是由于删除了它的标签或其他相关的镜像导致的。

无效的 none 镜像在镜像列表中以 "<none>" 来标记。这些镜像占据了磁盘空间，但通常没有实际用途，因此建议将其清理掉。
# 查看是否有无效的 none 镜像
docker images -f "dangling=true"
# 如果有的话就执行以下删除命令
docker rmi $(docker images -f "dangling=true" -q)
```



#### Docker 安装软件步骤

搜索镜像，拉取镜像，查看镜像，启动镜像，停止容器，移除容器 

安装MySQL镜像时注意编码问题，show variables like 'character%'(查看编码方式)，数据备份（容器数据卷）

解决编码格式新建my.cnf文件，配置编码格式（vim /docker/mysql/conf/my.cnf）

````my.cnf
[client] 
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8）
````

```console
docker run --name some-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag（基础版mysql）
docker run --name some-mysql -p 3306:3306 -e --privileged=true
-v /docker/mysql/log:/var/log/mysql
-v -v /docker/mysql/data:/var/lib/mysql
-v /docker/mysql/conf:/etc/mysql/conf.d
MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

安装redis镜像时需要注意的问题：容器卷，指定redis配置文件，copy 文件redis.conf 到本机目录文件夹下

````
docker run -p 6379:6379 --name myredis 
-v /usr/local/docker/redis/redis.conf:/etc/redis/redis.conf 
-v /usr/local/docker/redis/data:/data 
-d redis redis-server /etc/redis/redis.conf //后台启动，读取指定的配置文件（/etc/redis/redis.conf）
--appendonly yes
````



#### Dockers容器卷

数据持久化。映射，容器内的数据备份+持久化到本地主机目录；共享；实时生效；本地主机和容器内相互的，两者保持一致。

docker run -it --privileged=true  -v /宿主机绝对路劲目录：/容器内目录:ro 镜像名 ro（read  onl y)是设置容器的读写权限

查看数据卷是否成功 docker inspect 容器ID ，查看mounts挂在情况

容器继承容器docker run -it --privileged=true  --volumes-from u

#### Docker 容器卷和绑定挂载的区别

```
绑定挂载从Docker最早的时候就可以用于数据持久化。绑定挂载将把一个文件或目录从主机上挂载到你的容器上，然后你可以通过其绝对路径来引用。
docker run -d -v /hostdata:/app/data myapp
宿主机上的/hostdata目录与容器的/app/data目录关联起来，容器可以在此目录中读取和写入宿主机上的文件。
```

```
卷是在Docker容器中添加数据保存层的一个很好的机制，特别是在你需要在关闭容器后保存数据的情况下。

Docker卷完全由Docker本身处理，因此与你的目录结构和主机的操作系统无关。当你使用卷时，在主机上的Docker存储目录中会创建一个新的目录，而Docker会管理该目录的内容。

docker volume create myvolume
docker run -d -v myvolume:/app/data myapp
数据卷myvolume与容器的/app/data目录关联起来，容器可以在此目录中读取和写入数据。
```

#### Docker Mysql 主从复制搭建

```
docker run -p 3307:3306 --name mysql-master
-v /mydata/mysql-master/log:/var/log/mysql
-v /mydata/mysql-master/data:/var/lib/mysql
-v /mydata/mysql-master/conf:/etc/mysql
-e MYSQL_ROOT_PASSWORD = root
-d mysql:5.7//版本
配置conf文件，在/mydata/mysql-master/conf新建my.conf文件，并配置，重启容器
进入mysql-master容器，master容器实例内创建数据同步用户

创建从mysql服务器
docker run -p 3308:3306 --name mysql-slave
-v /mydata/mysql-master/log:/var/log/mysql
-v /mydata/mysql-master/data:/var/lib/mysql
-v /mydata/mysql-master/conf:/etc/mysql
-e MYSQL_ROOT_PASSWORD = root
-d mysql:5.7//版本
配置conf文件，在/mydata/mysql-master/conf新建my.conf文件，并配置，重启容器

回到主服务器查看主从同步状态show master status;
从数据库中配置主从配置
在从服务器中查看主从同步状态 
在从数据库中开启主从同步
```

```
Windows自己配置MySQL主从复制
先在本地创建my.cnf配置文件，目录如挂载的绝对路径；分别创建主从数据库的配置文件，master和slave;
创建主MySQL数据库
docker run -itd --name=master -p 3307:3306 //itd 后台运行
-e MYSQL_ROOT_PASSWORD=root
-v /d/demo/mysql/master/conf/my.cnf:/etc/mysql/my.cnf //直挂载了配置文件的目录
-d mysql:5.7
创建从Mysql数据库
docker run -itd --name=slave -p 3308:3306 //itd 后台运行
-e MYSQL_ROOT_PASSWORD=root
-v /d/demo/mysql/slave/conf/my.cnf:/etc/mysql/my.cnf //直挂载了配置文件的目录
-d mysql:5.7
进入主Mysql容器内部，创建从数据库的权限
CREATE USER 'slave'@'%' IDENTIFIED WITH 'mysql_native_password' BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
FLUSH PRIVILEGES; 
然后查看master状态
show master status;
进入从Mysql容器内部，配置从数据库
change master to master_host='172.19.128.1',master_user='slave',master_password='123456',
master_log_file='mysql-bin.000007',master_log_pos=1564,
master_port=3307, master_connect_retry=30;

master_host :主数据库IP地址，master_port:主数据库运行端口，master_user：在主数据库创建的用于同步数据的账号；
master_password：在主数据库创建的用于同步数据的密码；master_connect_retry：连接失败重连时间间隔，时间为秒；
master_log_file：指定从数据库要复制的数据的日志文件，通过查看主数据库的状态，获取到的File参数
master_log_pos：指定从数据库要复制的数据的日志文件，通过查看主数据库的状态，获取到的Position参数
开启复制
start slave;
查看状态
show slave status\G;如下图红线为Yes为开启了；


在实现主从复制时若出现[Warning] World-writable config file ‘/etc/mysql/conf.d/my.cnf‘ is ignored.
需要设置my.cnf 文件的权限，使用命令chmod 644 my.cnf更改权限，重启容器即可。
```

![image-20230627211346191](Docker.assets/image-20230627211346191.png)

![image-20230627211704221](Docker.assets/image-20230627211704221.png)

#### Docker 安装redis集群（3主3从redis集群）

```
方法：哈希取余分区（算法），一致性哈希算法分区，哈希槽分区（一个集群有16384个槽）
启动6台redis容器
1:
docker run -d --name redis-node-1 --net host --privileged=true
-v /data/redis/share/redis-node-1:/data redis:6.0.8
--cluster-enabled yes --appendonly yes --port 6381
2:
docker run -d --name redis-node-2 --net host --privileged=true
-v /data/redis/share/redis-node-2:/data redis:6.0.8
--cluster-enabled yes --appendonly yes --port 6382
3:
docker run -d --name redis-node-3 --net host --privileged=true
-v /data/redis/share/redis-node-3:/data redis:6.0.8
--cluster-enabled yes --appendonly yes --port 6383
4:
docker run -d --name redis-node-4 --net host --privileged=true
-v /data/redis/share/redis-node-4:/data redis:6.0.8
--cluster-enabled yes --appendonly yes --port 6384
5:
docker run -d --name redis-node-5 --net host --privileged=true
-v /data/redis/share/redis-node-5:/data redis:6.0.8
--cluster-enabled yes --appendonly yes --port 6385
6:
docker run -d --name redis-node-6 --net host --privileged=true
-v /data/redis/share/redis-node-6:/data redis:6.0.8
--cluster-enabled yes --appendonly yes --port 6386
进入容器1，执行命令构建主从关系
redis-cli --cluster create 192.168.2.252:6381 192.168.2.252:6382 192.168.2.252:6383 192.168.2.252:6384 192.168.2.252:6385 192.168.2.252:6386 --cluster-replicas 1
查看集群状态
redis-cli -p 6381
cluster info
集群检查
redis-cli --cluster check 192.168.2.252:6381
```

```
Redis集群扩容
新建6387和6388两个容器，并启动
进入6387容器内部，将新增的6387作为master节点加入集群
redis-cli --cluster add-node 192.168.2.252:6387 192.168.2.252:6381
检查集群情况（是否能查到6387）
redis-cli --cluster check 192.168.2.252:6381
重新分配槽号(由前几个容器匀出来给到新的容器)
redis-cli --cluster reshard 192.168.2.252:6381
为主节点6387分配从节点6388
redis-cli --cluster add-node 192.168.2.252:6388 192.168.2.252:6387
--cluster-slave --cluster-master-id (复制6387的容器ID)
```

```
Redis集群缩容
先将从节点6388清除
redis-cli --cluster del-node 192.168.2.252:6388 (填写6388容器ID)
将6387的槽号清空
redis-cli --cluster reshard 192.168.2.252:6381 (运行后将6387的槽号指定给哪个容器，填写容器ID)
集群检查，查看是否成功
redis-cli --cluster check 192.168.2.252:6381
将6387容器节点删除
redis-cli --cluster del-node 192.168.2.252:6387 (填写6387容器ID)
```

```
三种方法的比较：哈希取余分区（算法），一致性哈希算法分区，哈希槽分区
哈希取余分区：取余hash 会使用特定的数据，比如说用户Id、手机号等，再根据节点的数量N通过公式：hash(key) % N计算出哈希值。我们就可以定位到将用户用户id为10的数据存入到master1 的服务器节点中。当我们获取数据时一样通过公式能定位到master1 主节点，这们就省去了遍历所有服务器的时间，从而大大提升了性能。
缺点分析：当增加或减少节点时，原来节点中的80%的数据会进行迁移操作，对所有数据重新进行分布。
		当某一个主节点出现网络异常或者宕机产生不可达现象时，会出现整个集群环境下可用服务器基数的变化(从三台变成两台)。从而导致整个集群不可用，因为涉及到的计算公式已经变化，造成数据失效、混乱等现象。
优势：客户端分片
	配置简单：对数据进行哈希，然后取余
劣势：	数据节点伸缩时，导致数据迁移
	迁移数量和添加节点数据有关，建议翻倍扩容
	
一致性哈希算法分区：将所有的数据当做一个token环，token环中的数据范围是0到2的32次方。然后为每一个数据节点分配一个token范围值，这个节点就负责保存这个范围内的数据。对每一个key进行hash运算，被哈希后的结果在哪个token的范围内，则按顺时针去找最近的节点，这个key将会被保存在这个节点上。
	一致性hash算法就是为了解决上面取余hash的问题，首先失效的最主要原因是取余的基数是根据集群中的节点数动态变化的！一致性hash算法把取余的基数固定为一个常数 232，这样可以做到以不变应万变。
扩容时：在进行扩容时，插入到某两个节点之间，同时数据影响也只有两个之间的数据受影响，数据迁移范围缩小很多。
优势：解决数据失效的问题
	不需要考虑横向扩容问题
	有很好的容错性和可扩展性。
	采用客户端分片方式：哈希 + 顺时针(优化取余)
	节点伸缩时，只影响邻近节点，但是还是有数据迁移
劣势：主要就是数据分布不均匀（数据倾斜），当某个节点宕机，它就需要下一个节点来提供服务。

哈希槽分区：一个 Redis Cluster包含16384（0~16383）个哈希槽，存储在Redis Cluster中的所有键都会被映射到这些slot中，集群中的每个键都属于这16384个哈希槽中的一个。按照槽来进行分片，通过为每个节点指派不同数量的槽，可以控制不同节点负责的数据量和请求数.
	集群使用公式slot=CRC16（key）/16384来计算key属于哪个槽，其中CRC16(key)语句用于计算key的CRC16 校验和。
优势：很容易添加或者删除节点：比如如果我想新添加个节点D, 我需要从节点 A, B, C中转移部分槽到D上即可.如果我像移除节点A,需要将A中得槽移到B和C节点上,然后将没有任何槽的A节点从集群中移除即可.
```

#### DockerFile 

```
DOkcerFile 是用来构建Docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本。
构建三步骤：编写dockerfile文件，docker build命令构建镜像，docker run依镜像运行实例。
```

```
DockerFile 常用保留字指令
FROM:指定基础镜像，用于构建当前镜像的基础环境。
MAINTAINER:指定镜像维护者的信息。
RUN:在镜像构建过程中执行命令。可以用于安装依赖、配置环境等操作。
EXPOSE:声明容器运行时需要监听的端口号。
WORKDIR:设置容器中的工作目录。
USER:指定运行镜像时的用户名或UID。
ENV:设置环境变量。
ADD:将本地文件或目录复制到镜像中。它还可以将远程URL下载并添加到镜像中。
COPY:将本地文件或目录复制到镜像中。
VOLUME:声明持久化存储的挂载点，用于在容器和主机之间共享数据。
CMD:设置容器启动后默认执行的命令，可以被Dockerfile中的ENTRYPOINT指令覆盖。
ENTRYPOINT:设置容器启动后默认执行的命令，可以与Dockerfile中的CMD指令结合使用。它的参数可以通过docker run命令传递。
```

```
DockerFile gin-vue-admin的DockFile 文件详解

#使用golang:alpine作为构建阶段的基础镜像，命名为builder。Alpine是一个轻量级的Linux发行版，golang:alpine镜像包含了Go语言的开发环境。
FROM golang:alpine as builder

#将工作目录设置为/go/src/github.com/flipped-aurora/gin-vue-admin/server，即Go项目的根目录。
WORKDIR /go/src/github.com/flipped-aurora/gin-vue-admin/server

#将当前目录下的所有文件复制到容器的工作目录中。这将包括Go项目的所有源代码文件、配置文件等。
COPY . .

#在构建阶段执行的一系列命令。这些命令用于配置Go的环境变量（设置Go模块、设置代理等）、整理依赖（go mod tidy）以及构建Go项目（go #build -o server）。最终会生成一个名为server的可执行文件。
RUN go env -w GO111MODULE=on \
    && go env -w GOPROXY=https://goproxy.cn,direct \
    && go env -w CGO_ENABLED=0 \
    && go env \
    && go mod tidy \
    && go build -o server .

#使用alpine:latest作为运行阶段的基础镜像。同样，Alpine是一个轻量级的Linux发行版。
FROM alpine:latest

#设置一个用于标识镜像维护者的标签。
LABEL MAINTAINER="SliverHorn@sliver_horn@qq.com"

#将工作目录设置为/go/src/github.com/flipped-aurora/gin-vue-admin/server，与构建阶段保持一致。
WORKDIR /go/src/github.com/flipped-aurora/gin-vue-admin/server

#从构建阶段的builder镜像中复制文件到当前目录。--from=0 参数，从前边的阶段中拷贝文件到当前阶段中，多个FROM语句时，0代表第一个阶段。
COPY --from=0 /go/src/github.com/flipped-aurora/gin-vue-admin/server/server ./
COPY --from=0 /go/src/github.com/flipped-aurora/gin-vue-admin/server/resource ./resource/
COPY --from=0 /go/src/github.com/flipped-aurora/gin-vue-admin/server/config.docker.yaml ./

#声明容器将会监听的端口号，这里设置为8888。
EXPOSE 8888
#设置容器启动时的默认入口点，最终生成两个镜像，以第二个为运行容器。
ENTRYPOINT ./server -c config.docker.yaml

此DockerFile中使用两个FROM：
	第一个FROM指令使用golang:alpine作为基础镜像，并构建一个包含Go应用可执行文件的镜像，命名为builder。这是构建阶段。
	第二个FROM指令使用alpine:latest作为基础镜像，并从前一个构建阶段的builder镜像中复制生成的Go应用可执行文件和其他相关文件。这是运行阶段。
```

#### Docker Network

```
常用的三种网络模式：host，bride,none
能做什么：
	容器间的的互联和通信以及端口映射，
	容器IP变动时候可以通过服务名直接网络通信而不受影响
网络模式：
	bridge：个容器都会被分配一个独立的网络命名空间，并且 Docker 会创建一个名为 "docker0" 的虚拟网桥。容器之间可以通过这个虚	拟网桥相互通信，同时也可以通过主机网络接口访问外部网络。这是默认的网络模式。
	host:在主机模式下，容器与主机共享网络命名空间，即容器使用主机的网络堆栈，不进行网络隔离。这样容器就可以直接使用主机的网络接	口，使得容器的网络性能更高，但也失去了容器间的网络隔离。
	none:容器不会分配任何网络资源，即容器没有网络接口，无法与外部网络通信。通常用于特定安全要求或只需本地访问的场景。
	container:容器模式允许多个容器共享同一个网络命名空间，这意味着这些容器可以像是在同一主机上运行一样直接通信，共享同一个 IP 	地址。容器模式适用于需要将多个容器组合在一起形成一个整体应用的场景。
	自定义网络：用户可以自行创建和配置这些网络。自定义网络可以用于实现更复杂的容器通信和网络隔离需求，同时也能帮助提供更灵活的网络		设置。
	
	
自定义网络例子：
	docker network create my_network
	docker run -d --name container1 --network my_network -p 8080:80 nginx
	docker run -d --name container2 --network my_network -p 8081:80 nginx
```

#### Docker Compose容器编排

```
Docker Compose是一个用于定义和运行多个Docker容器的工具。它使用YAML文件来配置应用程序的服务、网络和卷等方面，并使用简单的命令进行容器编排。
Docker-compose 一文件:docker-compose.yml，两要素：服务和工程
创建Compose文件：在项目的根目录下创建一个名为docker-compose.yml的文件。这个文件将包含您应用程序的各个服务的配置信息。您可以在该文件中定义多个服务，每个服务代表一个容器。
定义服务：在Compose文件中，您可以定义每个服务的配置。这些配置包括镜像、环境变量、端口映射、卷挂载等。

示例文件：
    version: '3'
    services:
      web:
        image: nginx:latest
        ports:
          - 8080:80
        volumes:
          - ./app:/usr/share/nginx/html
        depends_on:
          - db
      db:
        image: mysql:5.7
        environment:
          - MYSQL_ROOT_PASSWORD=secret
          - MYSQL_DATABASE=myapp
```

