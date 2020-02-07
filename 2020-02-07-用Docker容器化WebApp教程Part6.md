# 前言：
本篇文章属于“用Docker容器化WebApp教程”中一系列文章中的 Part6 篇。

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
本节我们以 `docker-demo:3` 为例，介绍如何将镜像上传到 [Docker Hub](https://hub.docker.com/)。


# 上传镜像：
1. 首先，我们要到 Docker Hub 上注册一个帐号，网址为：[https://hub.docker.com/signup](https://hub.docker.com/signup)。
2. 在 Docker Hub 上创建一个镜像仓库（repository），本文中我们的仓库名就是 `docker-demo`。
3. 执行命令 `docker login` ，输入自己的用户名（Docker ID）和密码完成登录，如下：

```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: valleygtc
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

4. tag 镜像并上传：

```bash
$ docker tag docker-demo:3 valleygtc/docker-demo:3
$ docker image push valleygtc/docker-demo:3
```


# 下载镜像：
```bash
$ docker pull valleygtc/docker-demo:3
```


# 参考：
- [Docker官方文档-Get started Part5-Sharing images on Docker Hub](https://docs.docker.com/get-started/part5/)