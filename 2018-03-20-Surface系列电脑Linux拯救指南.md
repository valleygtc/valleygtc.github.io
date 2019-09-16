# Surface系列电脑运行Linux操作系统的小毛病：
微软出的Surface pro和Surface Book系列电脑运行Linux系统，都会出现很多问题。

个人情况：Surface pro3，安装Ubuntu、Deepin系统都有毛病：Wifi不稳定，而且一断就再也连不上了，之后再重启都会卡死在logo界面，只能长按电源键强制关机。网不稳定的时候Wifi最容易断。

# 解决记录：
一、
- 参考资料：https://winaero.com/blog/how-to-install-linux-on-surface-pro-3/

以为是网卡驱动的问题，于是更换新的网卡驱动，如下：

```bash
$ git clone git://git.marvell.com/mwifiex-firmware.git
$ mkdir -p /lib/firmware/mrvl/
$ cp mwifiex-firmware/mrvl/* /lib/firmware/mrvl/
```

失败。依然没解决Wifi的毛病。

二、
参考资料：
- https://www.zhihu.com/question/28193155
- https://github.com/jakeday/linux-surface
- https://wiki.archlinux.org/index.php/Talk:Microsoft_Surface_Pro_3

如果不是网卡驱动，那么就应该是内核问题（Linux内核对Surface系列的硬件支持不太好？）。于是在网上寻找专为surface整理的Linux内核，换一个专用内核即可：
见：https://github.com/jakeday/linux-surface
步骤如下：
(1) 准备工作：
clone下Linux的源代码（大概20多G），我这里使用的中国科技大学的镜像：

```bash
> mkdir ~/fix
> cd ~/fix
> mkdir linux && cd linux
> git init
#fetch比clone要好，因为其支持断点续传
> git fetch --tags  git://mirrors.ustc.edu.cn/linux.git
> git checkout FETCH_HEAD
> git checkout v4.15.10
```

clone下网上大神整理好的patch：

```bash
> cd ~/fix
> git clone https://github.com/jakeday/linux-surface.git
```

(2)
自己打patch并编译内核（我编译了大概两个小时）：

```bash
> cd ~/fix/linux
> sudo apt-get install build-essential binutils-dev libncurses5-dev libssl-dev ccache bison flex
> for i in ~/fix/linux-surface/patches/[VERSION]/*.patch; do patch -p1 < $i; done
> cp ~/fix/linux-surface/config .config
#注：这里不知道为什么必须要用sudo，我第一次没有用sudo，在编译完成整理成.deb安装包时异常中断了，提示什么权限不够只能再重新来过。
> sudo make -j `getconf _NPROCESSORS_ONLN` deb-pkg LOCALVERSION=-linux-surface
```

(3)
安装编译好的kernel和header：

```bash
> cd ~/fix
> sudo dpkg -i linux-headers-[VERSION].deb linux-image-[VERSION].deb
```

然后重启即可。
可以看一下已经是新的内核了：

```bash
> uname -r                                     
4.15.10-surface-linux-surface
```


# 结果：
成功修好了Wifi，再也不断了。但是出现了一个新问题：一睡眠就睡死过去了，无法唤醒，只能强制关机重启才可以。只好把Linux调成了永不睡眠状态。

# 其他：
Arch好像有人整理了surface的package来解决Linux kernel对surface系列硬件的东西，貌似不用自己编译了？见：https://aur.archlinux.org/packages/linux-surfacepro3-git/

