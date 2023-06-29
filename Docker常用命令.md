#### Docker 进程

```
#启动docker
service docker start
#关闭docker
service docker stop
#重启docker
service docker restart
```

#### Docker 镜像操作

```
#查看镜像列表
docker images（-a 查看所有镜像，-q 只显示镜像ID）
#搜索镜像
docker search 镜像名称
#拉取镜像
docker pull 镜像名称（可以指定镜像版本，不指定版本默认最新）
#删除镜像
docker rmi 镜像名称
#创建镜像
docker build -t 镜像的名字及标签，通常 name:tag 或者 name 格式；
#镜像中存在<none><none>的镜像,分为有效None和无效None
#删除无效None镜像
# 查看是否有无效的 none 镜像
docker images -f "dangling=true"
# 如果有的话就执行以下删除命令
docker rmi $(docker images -f "dangling=true" -q)
```

#### Docker 容器操作

```
#查看容器列表
docker ps(-a 查看所有容器,-q 只显示容器ID，-p 主机端口映射到docker端口)
#创建容器但不启动(docker create [options] image [command])
docker create -d --name redis-demo -p 6379:6379 redis:7.0.5
#根据某个镜像创建容器并启动（-d 后台运行,--cpus cpu核心数，-m指定最大内存，-e设置环境变量）
docker run -d --name redis-demo -p 6379:6379 --cpus 1 -m 100m -e REDIS_NAME-redis-demo redis:7.0.5
#以交互的方式进入容器
docker exec -it redis-demo /bin/bash
#启动容器
docker start redis-demo
#停止容器
docker stop redis-demo
#暂停容器
docker pause redis-demo
#运行状态下重启容器
docker restart redis-demo
#删除容器
docker rm redis-demo

#根据容器的当前变更反向生成镜像(docker commit [options] container [Reposiory:[tag]])
docker commit -m 'Image from container' redis-demo local-redis-image:1.0.0
#查看新建镜像的提交信息
docker inspect local-redis-image:1.0.0 | grep Comment
或docker image inspect local-redis-image:1.0.0 | grep Comment
#查看容器的详细信息
docker inspect redis-demo
#查看容器日志(docker logs [options] 容器名称(container))
docker logs redis-demo
```

#### Docker 存储操作（数据卷）（将容器卷与主机进行挂载）

```
#查看volume
docker volume ls
#创建volume
docker volume create v-redis
#删除volume
docker volume rm v-redis

#创建容器时绑定挂载数据卷(volume)，以mysql为例子
#docker run -it --privileged=true  -v /宿主机绝对路劲目录：/容器内目录:ro 镜像名 ro（read  onl y)是设置容器的读写权限
#其中:io可选 -v实现宿主机目录与容器内目录挂载(绑定挂载)
docker run --name some-mysql -p 3306:3306 -e --privileged=true
-v /docker/mysql/log:/var/log/mysql			//绑定挂载
-v -v /docker/mysql/data:/var/lib/mysql		//绑定挂载	
-v /docker/mysql/conf:/etc/mysql/conf.d		//绑定挂载
MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

#数据卷挂载，容器与创建的数据卷进行挂载
docker volume create myvolume
docker run -d -v myvolume:/app/data myapp
```

