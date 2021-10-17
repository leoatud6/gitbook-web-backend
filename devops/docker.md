---
description: 'Build once, run anywhere::: Image / container / repository'
---

# Docker

#### [https://www.yuque.com/leifengyang/oncloud/mbvigg](https://www.yuque.com/leifengyang/oncloud/mbvigg)

## Docker基本概念 （尚硅谷 云原生）

  docker build --> 变成一个generic的构建包。每个软件有自己的image。

 docker run --> 针对所有的镜像，直接run

 虚拟化virtualization 技术。-->容器化，虚拟隔离，内存泄露不会影响其他的。

![](<../.gitbook/assets/image (351).png>)

![ docker architecture](<../.gitbook/assets/image (352).png>)

{% embed url="https://hub.docker.com/_/registry" %}

{% embed url="https://www.yuque.com/leifengyang/oncloud/mbvigg" %}

```bash
sudo yum install -y docker-ce docker-ce-cli containerd.io
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
systemctl enable docker --now

docker info
docker ps

docker pull redis

docker run --name=mynginx -d nginx
# running in the background
docker run --name=mynginx -d --restart=always nginx
# auto restart whent the machine boot
docker run --name=mynginx -d --restart=always -p 88:80 nginx
# map the port

docker stop [id]
docker rm [id]
docker start [id]
curl 127.0.0.1:80
```

![](<../.gitbook/assets/image (353).png>)

* Docker Image: source image, mysql, redis, ES...
* container: after build, have small environment

![](<../.gitbook/assets/image (144).png>)

## Operation

```bash
docker run -d -p 80:80 --name webserver nginx
docker run --name=mynginx   \
-d  --restart=always \
-p  80:80 -v /data/html:/usr/share/nginx/html:ro  \
nginx

#Access localhost
docker stop webserver
docker rm webserver

docker pull ubuntu:18.04
docker run -it --rm \ubuntu:18.04 \bash

docker image ls
docker system df
docker image prune
docker image ls xx
docker image rm
docker run --name webserver -d -p 80:80 nginx
docker diff webserver

#将image放在container里面去run
Docker run -d -p 91:80 nginx:1.14  

docker exec -it f2281cff8c09 /bin/bash

docker commit --help
docker commit -a "chenliu" -m "first commit" f2281cff8c09 googlenginx:v1.0
# -a author
# -m message
# commit to lcoal docker images

docker save -o abc.tar 6a7188c9650e
# save the image to tar file so that can run in everywhere

docker load -i abc.tar
# load docker tar file

# SHARE: upload to docker hub, so that others can use.
docker tag --help
# docker tag first and then docker push
# like github, must have account first?!
docker login
docker logout

docker push
# upload the docker file, like github 


docker run redis redis-server /usr/local/etc/redis/redis.conf


docker run -v /data/redis/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data \
-d --name myredis \
-p 6379:6379 \
redis:latest  redis-server /etc/redis/redis.conf
# must create the corresponding path in the local machine and then map/mount
```

## Docker打包（dockerfile）

 需要有一个基础的运行环境，** openJDK** --> must be linux environment

Dockerfile (script file)

```bash
FROM openjdk:8-jdk-slim
# docker pull from the docker hub for the open jdk env
# from是基础镜像，env
LABEL maintainer=leifengyang

COPY target/*.jar   /app.jar
# copy the jar file into the container first

ENTRYPOINT ["java","-jar","/app.jar"]
```

```bash
docker build -t java-demo:v1.0 .
# . mean running in the current path

```

## 构建DockerImage

用Dockerfile构建： 

FROM nginx RUN echo <…content can print out…> >/usr/share/nginx/html/index.html 

Docker build -t nginx:chenliu . // 后面代表名字，注意后面有目录文件

_应用：将war包放进docker里面run，就是web application, 就可以看网页了web嘛_
