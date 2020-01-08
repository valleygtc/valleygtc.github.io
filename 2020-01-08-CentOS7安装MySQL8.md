# 安装
MySQL 被 Oracle 收购后，CentOS7之后的官方仓库中就不再提供 MySQL，而是提供其开源版本 MariaDB。<br>
MariaDB 的安装方法很简单：
```bash
# server端:
$ sudo yum install mariadb-server

# client端：
$ sudo yum install mariadb
```

若要安装 MySQL，需要添加 MySQL 社区提供的仓库：<br>
首先在[这里](https://dev.mysql.com/downloads/repo/yum/)找到对应版本的 `.rpm` 文件的下载链接。<br>
以 CentOS7，MySQL8 为例，执行命令：

```
$ wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
$ yum install mysql80-community-release-el7-3.noarch.rpm

$ yum install mysql-community-server
```

但是因为这个仓库下载很慢，我们最好使用镜像源下载，这里我们以使用清华的 MySQL 镜像源为例：
```bash
$ cd /etc/yum.repos.d/
$ touch tsinghua-mysql-community.repo
# 填入以下内容：
$ cat tsinghua-mysql-community.repo
[tsinghua-mysql80-community]
name=MySQL 8.0 Community Server tsinghua mirror.
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/
enabled=1
gpgcheck=0

# 然后安装：
$ yum install mysql-community-server
```

# 配置
```bash
# 启动mysql
$ systemctl start mysqld.service
```

MySQL8 中的 root 密码不再是默认为空，而是在启动时随机生成一个密码，mysqld 会将其写到日志文件中，所以首先我们需要找到这个密码：
```
$ cat /var/log/mysqld.log | grep 'password'
2020-01-08T03:43:14.175884Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: gerN_Zu=?9Pn
```

MySQL 提供了一个 bash 脚本 `mysql_secure_installation`，方便我们交互式的对MySQL进行一些基本的安全方面的设置：
1. 可以设置root密码
2. 可以禁止远程以root身份登录
3. 可以删除匿名用户
4. 可以删除test数据库，默认这个test数据库是可以被所有匿名用户访问的。

```
$ mysql_secure_installation
```

配置完成后应当重启 mysqld：
```
$ systemctl restart mysqld.service
```
