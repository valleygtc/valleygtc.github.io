# 前言
本篇文章属于“用Docker容器化WebApp教程”中一系列文章中的 Part3 篇。

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


# Docker 网络类型简介
Docker 的虚拟化网络有多种类型：bridge，host，overlay，macvlan，none 等，不同类型的网络功能与用处有所不同。

关于何时使用哪种类型的网络，简要概括如下：
- `bridge`：如果在启动容器时不指定网络，默认会连接到一个 bridge 类型的默认网络。如果我们想要实现在同一宿主机上的多个容器之间的通信，一般会自己创建一个 bridge 类型的网络，将容器连到其上。
- `host`：在 host 网络下运行的容器会和宿主机使用同一个网络栈。注意只有 Linux 支持此种类型，Windows 与 Mac 不支持。见[文档](https://docs.docker.com/network/host/)
- `overlay`：如果我们想要实现在不同宿主机上的多个容器之间的通信，使用这种类型的网络。
- `macvlan`：在 macvlan 类型的网络中，每一个容器都有自己的 MAC 地址，Docker daemon 会根据 MAC 地址来决定将网络数据转发到哪个容器。
- `none`：如果我们的容器不需要网络，那么就使用这个。


# 网络管理
Docker 在启动时会自动创建 3 个网络：bridge，host 和 none，如下：
```
# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
69791a0e50ab        bridge              bridge              local
f89815bfb316        host                host                local
764385936d61        none                null                local
```

docker 创建虚拟网络其实是使用更改操作系统的 `iptables` 规则来实现的，比如我们可以使用 `ip a` 命令来查看本机 ip：
```
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:02:97:0e brd ff:ff:ff:ff:ff:ff
    inet 172.26.37.34/28 brd 172.26.37.47 scope global noprefixroute dynamic eth0
       valid_lft 86163sec preferred_lft 86163sec
    inet6 fe80::80c4:f8d7:bca9:8744/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:e1:a2:a0:71 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e1ff:fea2:a071/64 scope link
       valid_lft forever preferred_lft forever
```

其中的 `eth0` 是我们本机在以太网的网络，可以看到在 `eth0` 网络中，本机的 ip 是 `172.26.37.34`。<br>
其中的`docker0` 就是 Docker 的默认 bridge 类型虚拟网络，是 Docker 启动时自动创建，在这个网络中，我们的 ip 是 `172.17.0.1`。

我们可以使用命令来创建虚拟网络，如下：
```
# docker network create foo_net
6e074a3d242edbc378e4db0114982f7c3046daaf95a3248ee4e7d7d611f53d45

# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
69791a0e50ab        bridge              bridge              local
6e074a3d242e        foo_net             bridge              local
f89815bfb316        host                host                local
764385936d61        none                null                local
```

我们使用 Docker CLI 来创建了一个 bridge 类型的名为 foo_net 的网络，我们再使用 `ip a` 命令来观察：
```
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:02:97:0e brd ff:ff:ff:ff:ff:ff
    inet 172.26.37.34/28 brd 172.26.37.47 scope global noprefixroute dynamic eth0
       valid_lft 86384sec preferred_lft 86384sec
    inet6 fe80::80c4:f8d7:bca9:8744/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:e1:a2:a0:71 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e1ff:fea2:a071/64 scope link
       valid_lft forever preferred_lft forever
104: br-6e074a3d242e: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:b7:50:19:2d brd ff:ff:ff:ff:ff:ff
    inet 172.22.0.1/16 brd 172.22.255.255 scope global br-6e074a3d242e
       valid_lft forever preferred_lft forever
```

可以看到，多出的一个 `br-6e074a3d242e` 就是我们自己创建的虚拟网络，在这个网络中，本机的 ip 是 `172.22.0.1`。


# 用户创建的 bridge 类型网络和默认 brige 类型网络
我们在启动容器时，使用 `--network` 参数来指定我们想要连接到的网络。<br>
bridge 类型是我们最常用的网络类型。在启动容器时，如果不声明 `--network` 参数，那么默认会连接到 Docker 的那个名为 bridge 的默认网络上。在该网络上，我们可以使用 ip 地址来访问到在此网络上的其他容器。

在实际使用中，一般我们都会自己创建 bridge 类型的网络，将容器连到其上。这种自己创建的 bridge 类型网络，相比于默认的 bridge 网络有很多的优点：
1. 自建 bridge 网络中，默认所有容器的端口对所有同一网络的其他容器开放，对外不开放。而在默认 bridge 网络中，必须使用 `-p` 参数将端口绑定到本机相应端口上，但是这样就自动对外界网络也开放了。
2. 自建 bridge 网络提供了 DNS 解析服务，我们可以直接使用容器名作为域名来访问其他容器。而默认 bridge 类型网络中，我们只能使用 ip 地址来访问其他容器。
3. 容器可以在运行时可以随时与网络连接或断开连接。而在默认的 bridge 网络上，需要停止然后重新创建容器来使用不同的网络参数。
等


# 参考：
- [docker network overview](https://docs.docker.com/network/)
- [Use bride networks](https://docs.docker.com/network/bridge/)
- [Use host networking](https://docs.docker.com/network/host/)
