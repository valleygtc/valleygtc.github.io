# 前言：
本篇文章属于“用Docker容器化WebApp教程”中一系列文章中的 Part5 篇。

文章列表：
- [用Docker容器化WebApp教程-Part1](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part1.md)
- [用Docker容器化WebApp教程-Part2](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part2.md)
- [用Docker容器化WebApp教程-Part3](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part3.md)
- [用Docker容器化WebApp教程-Part4](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part4.md)
- [用Docker容器化WebApp教程-Part5](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part5.md)
- [用Docker容器化WebApp教程-Part6](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-02-07-用Docker容器化WebApp教程Part6.md)


项目的源代码可以在 [github](https://github.com/valleygtc/docker-demo) 上获取：
```bash
$ git clone https://github.com/valleygtc/docker-demo
```

我的博客原文放在 github 上，[这里](https://github.com/valleygtc/valleygtc.github.io)。


# 简介
在上一篇文章中我们给 docker-demo 加上了数据库，并使用 docker CLI 来启动 mysql 和 docker-demo 容器。<br>
这种做法有一个问题，就是每次启动应用都需要我们手动输入命令，如果启动多个容器就需要手动输入多次命令，繁琐且也容易出错。<br>
我们可以使用 `docker-compose` 这个工具来解决这个问题。

docker-compose 的简单介绍：<br>
一个应用程序通常需要与许多外部软件共同协作来运行，如一个 Web app 通常需要 MySQL，Redis，Nginx 等。docker-compose 允许我们使用 `docker-compose.yaml` 文件来声明配置，帮助我们同时运行多个容器。除此之外它还有许多其他功能，如可以帮助我们创建网络等。


# 安装 docker-compose：
各系统安装方法具体可见[文档](https://docs.docker.com/compose/install/)，在这里我将安装步骤列举如下：

Windows 和 Mac 系统中 Docker Desktop 就已经包含了 docker-compose 了，不需要单独安装。

Linux 下需要单独安装，方法如下：
```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
```

Github 访问太慢，我们可以使用国内镜像 [DaoCloud](https://get.daocloud.io/#install-compose) 来下载，方法如下：
```bash
$ curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

注意：下载下来记得校验 SHA256，和官方在 [Github-release](https://github.com/docker/compose/releases) 上发布的的对比，确保文件完整且未被篡改。

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

在启动之前，我们需要将之前启动的正在运行的 `docker-demo` 和 `mysqd` 停止：
```bash
$ docker container stop docker-demo mysqd
```

然后，使用 docker-compose 启动：
```
$ docker-compose up
```

验证程序是在正常运行：
```bash
$ curl 'http://127.0.0.1:5000/student/'
{"data":[{"address":null,"age":22,"id":1,"name":"gutianci"}],"success":true}
```

解释：<br>
如果我们没有在 yaml 文件中声明网络，在使用 `docker-compose up` 启动服务时 docker-compose 默认会自动帮我们创建一个名为 `{project}_default` 的 bridge 类型的网络，并将所有的容器连入此网络。如下：
```
# docker network ls
NETWORK ID          NAME                  DRIVER              SCOPE
69791a0e50ab        bridge                bridge              local
aa52d19b5913        docker-demo_default   bridge              local
f89815bfb316        host                  host                local
764385936d61        none                  null                local
```

可以看到其中的 `docker-demo_default` 网络就是 docker-compose 自动帮助我们创建的。

如果需要，我们也可以在配置文件中自定义网络，让 docker-compose 在启动应用时帮我们创建。如下：
```yaml
version: "3.7"
services:
  docker-demo:
    image: docker-demo:3
    networks:
      - foo_net
    environment:
      - FLASK_ENV=production
      - DATABASE_URI=mysql+mysqlconnector://root:foopassword@mysqld/docker_demo?charset=utf8 # WARNING: You should change it!
    ports:
      - "5000:5000"
    volumes:
      - ./log:/docker-demo/log
  mysqld:
    image: mysql:8.0
    networks:
      - foo_net
    environment:
      - MYSQL_ROOT_PASSWORD=foopassword # WARNING: You should change it!
      - MYSQL_DATABASE=docker_demo
    volumes:
      - ./data:/var/lib/mysql
networks:
  foo_net:
    driver: bridge
```

该文件的配置与我们在上篇文章中使用的命令行参数是等价的，创建一个名为 `foo_net` 的 bridge 类型网络，启动 `docker-demo` 和 `mysqld` 两个容器并将他们接入此网络。


# docker-compose 常用命令：
- `docker-compose up`：Create and start containers
    - `-d/--detach`：默认是在前台运行，使用此参数后会在后台运行。
- `docker-compose down`: Stops containers and removes containers, networks, volumes, and images created by up.
- `docker-compose start [SERVICE...]`：Start services
- `docker-compose stop [SERVICE...]`：Stop services
- `docker-compose ps`：List containers

- `docker-compose run [OPTIONS] SERVICE [COMMAND] [ARGS...]`: Run a one-off command

- `docker-compose config`：Validate and view the Compose file


# Bonus：容器自动重启策略
我们可以在启动容器时使用 `--restart` 参数来设置重启策略，有四个值可供选择：
- `no`：不自动重启，默认值。
- `on-failure`：只有在程序异常退出时（程序返回值为非零）才会自动重启。
- `always`：总是自动重启。
- `unless-stopped`：总是自动重启，除非之前手动将其停止了。

使用这个参数我们就可以实现“程序挂掉自动重启”和“容器开机自启”的功能。

其中需要注意，因为容器的自动重启策略其实是由 Docker daemon 来管理的，所以如果想要实现容器开机自启，那么首先我们一定要将 Docker daemon 服务加入开机自启：
```
$ systemctl enable docker
```

具体见文档：[Start containers automatically](https://docs.docker.com/config/containers/start-containers-automatically/)

# 参考：
- [docker-compose官方文档](https://docs.docker.com/compose/)
- [文档：Networking in Compose](https://docs.docker.com/compose/networking/)
- [文档：compose file reference](https://docs.docker.com/compose/compose-file/)
