# 前言：
本篇文章属于“用Docker容器化WebApp教程”中一系列文章中的 Part4 篇。

文章列表：
- [用Docker容器化WebApp教程-Part1](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part1.md)
- [用Docker容器化WebApp教程-Part2](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part2.md)
- [用Docker容器化WebApp教程-Part3](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part3.md)
- [用Docker容器化WebApp教程-Part4](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part4.md)


项目的源代码可以在 [github](https://github.com/valleygtc/docker-demo) 上获取：
```bash
$ git clone https://github.com/valleygtc/docker-demo
```

我的博客原文放在 github 上，[这里](https://github.com/valleygtc/valleygtc.github.io)。


# 简介
在上一篇文章中我们给 docker-demo 加上了数据库。并使用 CLI 命令行启动 mysql 和 docker-demo 容器。<br>
有一个问题，就是每次启动应用都需要我们手动输入命令，如果启动多个容器就需要手动输入多次命令，太繁琐了，而且也容易出错。<br>
我们可以使用 `docker-compose` 这个工具来解决这个问题。

docker-compose 简单介绍：<br>
一个应用程序通常需要与许多外部软件共同协作来运行，如一个 Web app 通常需要 MySQL，Redis，Nginx 等。docker-compose 允许我们使用 `docker-compose.yaml` 文件来声明配置，帮助我们同时运行多个容器。除此之外它还有许多其他功能，如可以帮助我们创建网络等。


# 安装 docker-compose：
各系统安装方法具体可见[文档](https://docs.docker.com/compose/install/)，在这里我将安装步骤列举如下：

Windows 和 Mac 系统中 Docker Desktop 就已经包含了 docker-compose 了，不需要单独安装。

Linux 下需要单独安装，方法如下：
```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
```

github 访问太慢，我们可以使用国内镜像 DaoCloud 来下载：见[这里](https://get.daocloud.io/#install-compose)
```bash
$ curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

注意：下载下来记得校验SHA256，和官方在[Github-release](https://github.com/docker/compose/releases)上发布的的对比，看看文件是否完整。

验证 docker-compose 是否可用：
```
$ docker-compose version
```


# 使用 docker-compose 启动容器：
首先，创建 `docker-compose.yaml` 文件，内容如下：
```yaml
version: "3.7"
services:
  docker-demo:
    image: docker-demo:3
    links:
      - mysqld
    environment:
      - FLASK_ENV=production
      - DATABASE_URI=mysql+mysqlconnector://root:foopassword@mysqld/docker_demo?charset=utf8 # WARNING: You should change it!
    ports:
      - "5000:5000"
    volumes:
      - ./log:/docker-demo/log
  mysqld:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=foopassword # WARNING: You should change it!
      - MYSQL_DATABASE=docker_demo
    volumes:
      - ./data:/var/lib/mysql
```

该文件的配置与我们在上篇文章使用的命令行参数是等价的。

在启动之前，我们需要将之前启动的正在运行的 `docker-demo` 和 `mysqd` 停止：
```bash
$ docker container kill docker-demo mysqd
```

然后，使用 docker-compose 启动：
```
$ docker-compose up
```

验证程序是在正常运行：
```bash
$ curl http://127.0.0.1:5000/student/
{"data":[{"address":null,"age":null,"id":1,"name":"gutianci"}],"success":true}
```


# 参考：
- [docker-compose官方文档](https://docs.docker.com/compose/)
