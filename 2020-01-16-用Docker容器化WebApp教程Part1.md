# 前言
在本系列文章中，我们将使用 Flask 写一个 Web app，从最简单开始，逐渐给它添加功能，在每一步我们都将其容器化，来演示 Docker 的使用。

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


# 简介：
在本篇文章中，我们将使用 Flask 写出一个最简单的 Web app，其功能只有一个，就是 GET 请求其根 URL，会返回一段文本：`Hello, World!`。<br>
我们使用 waitress 服务器来部署这个应用。然后将其容器化。


# docker-demo 实现：
项目代码：
```bash
$ cd docker-demo
$ git checkout 1
```

该应用的目前依赖为：Flask 和 waitress。

项目的 `app.py` 文件内容如下：
```python
from flask import Flask


app = Flask(__name__)


@app.route('/')
def handle_hello():
    return 'Hello, World!'
```

`run.py` 文件内容如下：
```python
import os

from waitress import serve

from app import app


port = os.getenv('PORT', 5000)
serve(app, listen=f'*:{port}')
```

注意我们的应用会从环境变量中读入 `PORT` 来决定监听哪个端口。

我们通过 `python run.py` 命令即可运行这个应用。


# 将 docker-demo 容器化：
在项目根目录我们有一个 `Dockerfile` 文件，内容如下：
```dockerfile
FROM python:3.7
LABEL maintainer="gutianci@pwrd.com"

# 将文件复制到 image 中。
COPY . /docker-demo

# pip 安装程序依赖
WORKDIR /docker-demo
RUN python3 -m venv .venv
RUN . .venv/bin/activate && pip install --no-cache-dir -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 声明我们的程序运行需要的环境变量
ENV PORT=5000

EXPOSE 5000
CMD . .venv/bin/activate && python run.py
```

我们使用以下命令来构建镜像：
```bash
$ docker image build --tag=docker-demo:1 .
```


# 运行：
```bash
$ docker run -p 5000:5000 -d docker-demo:1
```

解释：
`docker run` 命令中的 `-p/--publish [HOST PORT]:[CONTAINER PORT]` 参数将宿主机的端口与容器的端口绑定，所有通过宿主机端口的数据都会自动转发到容器的端口上。注意，其实我们在 Dockerfile 中的 `EXPOSE 5000` 并没有真正的将容器端口与宿主机端口绑定，它仅仅是一个文档，镜像的构建者通过它来告诉镜像的使用者该镜像有监听哪些端口。我们也可以使用 `-P` 参数直接将容器所有的监听端口全部绑定到宿主机上。


检查程序是否正常对外提供服务：
```bash
$ curl http://127.0.0.1:5000/
```

我们应该得到响应：
```bash
Hello, World!
```


# --network=host mode
Docker 在关于容器的网络虚拟化方面有多种模式（network mode）：bridge，host，overlay，macvlan，none等。详细见：[Docker文档-network部分](https://docs.docker.com/network/)。

如果没有显式声明，默认是使用 `bridge` 模式来运行容器，在该模式下 Docker 会创建一个默认名为 `docker0` 的网桥，宿主机和容器的网络是隔离开的，宿主机和每一个容器都有各自的 IP，这也是我们之所以需要将端口绑定在一起的原因。
我们也可以使用 `host` 模式来运行容器，在此模式下容器会和宿主机使用同一个网络栈。因此，所有容器监听的端口都会直接绑定到宿主机上（因为二者使用的是同一个网络栈），`-p` 参数无需声明，即使是声明了也会被忽略。

因此我们也可以直接使用 host 模式来运行容器：
```bash
$ docker run --network=host -d docker-demo:1
```

注意：只有 Linux 系统才支持 host 模式，Windows 与 Mac 不支持。
