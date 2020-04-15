---
title: docker+node记录
date: 2020-03-06 11:36:45
tags:
	- node.js
---
docker+node 快速移植应用，隔离应用
<!--more-->

> 前言

docker可以实现快速部署应用，持续构建，隔离多个应用，快速迁移应用。docker+node可以实现多个应用运行在同一台服务器上，并且相互隔离，
互不影响。

> Dockerfile

首先搭建个简单的node应用, 可以通过`localhost:8080`输出`helloworld`即可。然后编写Dockerfile创建镜像

```Dockerfile
FROM node:12.16.1-alpine    

RUN mkdir -p /home/node
COPY . /home/node/
WORKDIR /home/node

RUN npm install --production

EXPOSE 8080
CMD ["node", "index.js"]

```

以上代码的意思是：

- 基于`node:12.16.1-alpine`node版本作为基础镜像;

- 创建个虚拟环境，并创建目录`/home/node`作为node
目录

- 把当前目录文件拷贝到`/home/node`

- 并以`/home/node`作为工作目录

- 安装npm依赖

- 暴露8080端口

- 运行node

这样便创建了一个node应用镜像，接下来需要通过docker-compose实例化一个容器。我们可以
通过docker-compose实现多个服务编排在一起，组成一个以node+MongoDB+redis`完全体`应用。

docker-compose.yml:
```dockerfile
version: '3'
services:
  node-docker:
    image: node-docker
    container_name: node-docker
    ports:
      - 8081:8080
    volumes:
      - ~/project/node-docker/logs:/home/node/logs
    networks:
      - node-docker-network
    environment:
      WAIT_HOSTS: database:27017, redis:6379

  database:
    image: mongo:4.2.0
    restart: always
    ports:
      - 27027:27017
    command: mongod --bind_ip 0.0.0.0
    volumes:
      - ~/data/node-docker-db:/data/db
    networks:
      - node-docker-network

  redis:
    image: redis:5.0.5
    restart: always
    networks:
      - node-docker-network

networks:
  node-docker-network:
    name: node-docker-network
```

以上代码我们通过编排任务将node应用和MongoDB和redis连接在同一个网络node-docker-network中，使node可以访问MongoDB和redis，database
就是mongo服务的别名，所以node访问时直接可以通过`mongodb://database`访问MongoDB。

`volumes`代表着数据卷的映射，当我们重新构建docker服务时，再次实例化镜像，会是一个全新的容器，上一个容器的数据卷会随着容器销毁。
使用volumes可以把对应的目录实时更新到宿主机目录中，再次实例化容器时，又会把宿主机的目录拷贝一份到新的容器目录中。

`WAIT_HOSTS`是一个wait脚本的变量，他代表在启动node应用之前应该等待的服务。即数据库服务应比node应用先启动。
所以应在Dockerfile中加上一句`CMD /scripts/wait`来执行这个脚本，即在数据库之后才去启动node应用容器

> 启动

那么怎么去运行compose呢——需要个脚本去启动：

ssh:

```ssh
docker build -t node-docker .
docker-compose up -d
```

这样，搭建一个docker+node+mongo的虚拟环境就完成啦！