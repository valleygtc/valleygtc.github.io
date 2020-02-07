# 前言
本篇文章属于“用Docker容器化WebApp教程”中一系列文章中的 Part2 篇。

文章列表：
- [用Docker容器化WebApp教程-Part1](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part1.md)
- [用Docker容器化WebApp教程-Part2](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part2.md)
- [用Docker容器化WebApp教程-Part3](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part3.md)
- [用Docker容器化WebApp教程-Part4](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part4.md)
- [用Docker容器化WebApp教程-Part5](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part5.md)
- [用Docker容器化WebApp教程-Part6](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part6.md)


项目的源代码可以在 [github](https://github.com/valleygtc/docker-demo) 上获取：
```bash
$ git clone https://github.com/valleygtc/docker-demo
```

我的博客原文放在 github 上，[这里](https://github.com/valleygtc/valleygtc.github.io)。


# 简介：
Docker 容器所接触的文件系统是与宿主机（host）隔离开的，容器无法直接读取本机的文件，也无法直接把数据写入本机。
即每一次容器运行，其文件系统都是新的。
如果我们要将容器内数据持久化到本机，需要将容器内的路径挂载到本机文件系统。

在本篇文章中，我们给 docker-demo 加上日志功能：
将每一次请求的 User-Agent 记录到文件 `log/access.log`中，然后使用 `-v/--volumn [HOST_DIR]:[CONTAINER_DIR]` 参数将容器内路径挂载到本机。


# docker-demo 实现：
项目代码：
```bash
$ git checkout 2
```

`app.py` 文件修改如下：
```python
import os
import logging
from logging import FileHandler

from flask import Flask, request


app = Flask(__name__)


# INFO -> log file
if not os.path.isdir('log'):
    os.mkdir('log')
app.logger.setLevel(logging.INFO)
fh = FileHandler('log/access.log', encoding='utf-8')
fh.setLevel(logging.INFO)
formatter = logging.Formatter('[%(asctime)s] %(levelname)s in %(module)s: %(message)s')
fh.setFormatter(formatter)
app.logger.addHandler(fh)


@app.route('/')
def handle_hello():
    ua = request.headers.get('User-Agent')
    app.logger.info('access from User-Agent: %s', ua)
    return 'Hello, World!'
```


# 将 docker-demo 容器化：
`Dockerfile` 文件不变，内容如下：
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

构建镜像：
```bash
$ docker image build --tag=docker-demo:2 .
```


# 运行：
```bash
$ docker run -p 5000:5000 -v "$PWD/log":/docker-demo/log -d docker-demo:2
```

解释：`-v "$PWD/log":/docker-demo/log` 参数的含义就是将容器内的 `/docker-demo/log` 挂载到宿主机的 `$PWD/log` 目录上。<br>
注意如果是在 Windows 下，需要注意两点：
1. 需要将要挂载的盘符共享，设置方法为：右键点击任务栏的 docker 图标 -> 进入 Settings -> Shared Drivers -> 勾选上要共享的目录。文档见[这里](https://docs.docker.com/docker-for-windows/#/shared-drives)。
2. Windows 下的路径分隔符为反斜杠，需要转义，如：`D:\\code\\demo\\docker-demo\\log`。

检查程序是否正常对外提供服务：
```bash
$ curl 'http://127.0.0.1:5000/'
```

我们应该得到响应：
```bash
Hello, World!
```

查看当前目录下的 `log` 子目录中的 `access.log` 文件，会有如下日志：
```bash
[2020-01-16 01:27:25,933] INFO in app: access from User-Agent: curl/7.61.1
```

证明我们的程序工作正常且 log 文件也保存到了本机上。
