# 前言
本篇文章属于“用Docker容器化WebApp教程”中一系列文章中的 Part4 篇。

文章列表：
- [用Docker容器化WebApp教程-Part1](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part1.md)
- [用Docker容器化WebApp教程-Part2](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part2.md)
- [用Docker容器化WebApp教程-Part3](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part3.md)
- [用Docker容器化WebApp教程-Part4](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part4.md)
- [用Docker容器化WebApp教程-Part5](https://github.com/valleygtc/valleygtc.github.io/blob/master/2020-01-16-用Docker容器化WebApp教程Part5.md)


项目的源代码可以在 [github](https://github.com/valleygtc/docker-demo) 上获取：
```bash
$ git clone https://github.com/valleygtc/docker-demo
```

我的博客原文放在 github 上，[这里](https://github.com/valleygtc/valleygtc.github.io)。


# 简介
在本篇文章中，我们将给 docker-demo web app 加上 MySQL 数据库。


# 使用 mysql Docker 镜像：
```bash
# 从镜像仓库拉取 mysql 8.0 镜像
$ docker pull mysql:8.0

# 首先，我们需要创建挂载目录：data。注意我们一定要确保在 mysql 启动时 data 目录就是存在的，以便 mysql 容器启动时完成初始化工作。
$ mkdir data

# 运行
$ docker run \
  --name=mysqld \
  -e MYSQL_ROOT_PASSWORD=foopassword \
  -e MYSQL_DATABASE=docker_demo \
  -v "$PWD/data":/var/lib/mysql \
  --rm -d mysql:8.0;
```

解释：
1. `-e MYSQL_ROOT_PASSWORD=foopassword`：我们使用 root 用户，密码为：`foopassword`。
2. `-e MYSQL_DATABASE=docker_demo`：我们使用仓库名为 `docker_demo`。
3. `-v "$PWD/data":/var/lib/mysql`：将数据目录挂载到本机的 `$PWD/data` 目录。

注：
1. 在 Windows 系统中 `$PWD` 可能会导致错误，需要手动输入绝对路径，如：`-v "D:\\code\\demo\\docker-demo\\data":/var/lib/mysql`。
2. 注意如果我们挂载的目录（即 `$PWD/data` 目录）中已经有数据库文件了，那么我们声明的所有环境变量都没有作用，MySQL 容器会直接使用已存在的数据库及用户。


我们使用以下命令连接到 mysqld 容器：
```bash
$ docker run -it \
  --link=mysqld \
  --rm mysql:8.0 \
  mysql -hmysqld -uroot -p;
```

解释：
1. 因为我们没有指定 `--network` 参数，此时容器连接到的是 bridge 默认网络，容器之间的端口并未互相开放，在这里我们使用 `--link=mysqld` 参数指示 Docker 将 docker-demo 与 mysqld 容器连接起来，这样，我们也可以直接使用容器名作为域名来访问 mysqld。

注：其实官方已经将 `--link` 参数标注为 **legacy** ，不建议使用。如果想要将实现容器间通信，推荐使用“user-defined networks”的方式。见[文档](https://docs.docker.com/network/links/)。但是我们在这里作为教程示例，简便起见，就这样用了。在接下来的我们自己的 docker-demo app 与 mysqld 的连接处理上，我们创建一个 bridge 类型网络来使用。


我们使用 root 用户登录，输入密码：`foopassword`，登录成功后我们可以看一下，`docker_demo` 数据库确实已经被自动创建了：
```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| docker_demo        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)
```

有一点需要注意：如果我们挂载的数据目录没有内容，mysql 在启动时会对其进行初始化，此时不对外提供连接服务，直至完成。所以在刚启动 mysqld 时可能无法连接上去，需要等一会才行。见[文档]((https://hub.docker.com/_/mysql))中的 “No connections until MySQL init completes”一节。

我们可以使用 `docker container logs` 命令查看其运行细节。


# docker-demo 实现：
项目代码：
```bash
$ git checkout 3
```

为了演示 web app 与数据库结合起来的用法，在 `app/model.py` 中我们建表：
```python
class Student(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), nullable=False)
    age = db.Column(db.Integer)
    address = db.Column(db.String(64))

    def readyToJSON(self, keys):
        """
        Params:
            keys [Iterable[str]]
        """
        d = {}
        for k in keys:
            v = getattr(self, k)
            d[k] = v
        return d

    def __repr__(self):
        return '<Student %r>' % self.id
```

我们实现了两个接口，
1. `GET /student/`：返回所有的学生
2. `GET /student/add?name=[str]&age=[int]&address=[str]`：添加学生。

在 `app/views.py` 文件中的代码如下：
```python
@bp_main.route('/student/', methods=['GET'])
def hanlde_show_students():
    records = Student.query.all()

    columns = ['id', 'name', 'age', 'address']
    return jsonify(
        success=True,
        data=[r.readyToJSON(columns) for r in records]
    )


"""
GET ?name=[str]&age=[int]&address=[str]
age, address is optional.
"""
@bp_main.route('/student/add', methods=['GET'])
def hanlde_add_student():
    name = request.args.get('name')
    if name is None:
        return jsonify(
            success=False,
            msg='You must assign a name field.'
        )

    age = request.args.get('age')
    if age is not None:
        age = int(age)

    address = request.args.get('address')    
    s = Student(name=name, age=age, address=address)
    db.session.add(s)
    db.session.commit()

    return jsonify(
        success=True,
        msg=f'Add student {name} success.'
    )
```


# 将 docker-demo 容器化：
我们的 `Dockerfile` 文件如下：
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
# 可以在 run 时覆盖
ENV FLASK_APP=manage.py
ENV FLASK_ENV=production
ENV DATABASE_URI='mysql+mysqlconnector://{user}:{password}@{localhost}/{db_name}?charset=utf8'
ENV PORT=5000

EXPOSE 5000
CMD . .venv/bin/activate && python run.py
```

解释：我们的程序需要 `FLASK_APP`，`FLASK_ENV`，`DATABASE_URI`，`PORT` 环境变量，`Dockerfile` 中的声明其实仅作为文档，我们可以在运行时声明这些环境变量。


构建镜像：
```bash
$ docker image build --tag=docker-demo:3 .
```


# 运行：
首先，我们需要创建一个 bridge 类型的网络，如下：
```bash
$ docker network create foo_net
```

停止之前的 mysqld 容器，重新创建一个新的 mysqld 容器，并将其连入 foo_net 网络：
```bash
$ docker container stop mysqld

$ docker run \
  --network=foo_net \
  --name=mysqld \
  -e MYSQL_ROOT_PASSWORD=foopassword \
  -e MYSQL_DATABASE=docker_demo \
  -v "$PWD/data":/var/lib/mysql \
  --rm -d mysql:8.0;
```

启动 docker-demo 并将其连入 foo_net 网络：
```bash
$ docker run \
  --network=foo_net \
  --name='docker-demo' \
  -e FLASK_ENV=production \
  -e DATABASE_URI='mysql+mysqlconnector://root:foopassword@mysqld/docker_demo?charset=utf8' \
  -p 5000:5000 \
  -v "$PWD/log":/docker-demo/log \
  --rm -d 'docker-demo:3';
```

我们的程序需要使用名为 student 的数据表，我们需要手动创建。<br>
首先，连接到运行中的 `docker-demo` 容器：
```bash
$ docker container exec -it 'docker-demo' bash
```

然后：
```bash
# 激活虚拟环境：
$ . .venv/bin/activate

# 创建表：
$ flask create_tables

# 退出容器
$ exit
```


# 验证程序正常运行：
```bash
$ curl http://127.0.0.1:5000/
Hello, World!

$ curl http://127.0.0.1:5000/student/
{"data":[],"success":true}

$ curl http://127.0.0.1:5000/student/add?name=gutianci&age=22
{"msg":"Add student gutianci success.","success":true}

$ curl http://127.0.0.1:5000/student/
{"data":[{"address":null,"age":null,"id":1,"name":"gutianci"}],"success":true}
```


# 参考：
- [DockerHub-mysql](https://hub.docker.com/_/mysql)
