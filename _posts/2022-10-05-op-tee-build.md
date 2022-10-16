---
title: op-tee安装构建（qemu）
tag: [tee]
key: 2022-10-05-op-tee-build
---



发现有op-tee开发者开源的docker：

[jbech-linaro/docker_optee: Simple Dockerfile that makes it easy to try OP-TEE using Docker (github.com)](https://github.com/jbech-linaro/docker_optee)

本人没有尝试，使用的手动编译，但是后来发现其中[Dockerfile-ubuntu-22.04-L14](https://github.com/jbech-linaro/docker_optee/blob/f82883937572daa94c45c6e24017ad712727c622/Dockerfile#L14)写的安装依赖包的命令更完善，可以**直接找到对应ubuntu版本的[Dockerfile](https://github.com/jbech-linaro/docker_optee/blob/ubuntu-22.04/Dockerfile)来尝试安装**。

# 环境配置

## 本机环境

使用的是wsl2，ubuntu20.04，阿里云和清华tuna源

## 依赖包安装

参照[Prerequisites — OP-TEE documentation](https://optee.readthedocs.io/en/latest/building/prerequisites.html)安装对应linux发布版的包

更换完国内源后输入以上安装包命令，报错：

>E: Unable to correct problems, you have held broken packages.

把`apt-get`更换为`aptitude`，解决报错

```bash
# 参考
sudo apt-get install aptitude
sudo aptitude install myNewPackage
```

## 安装repo

参考[安装repo-Android开源](https://source.android.com/docs/setup/develop#installing-repo)中的命令

如果是手动安装需要把最后一个命令的`~/bin/repo`更改为`/usr/local/bin/repo`，如下，才能直接在bash里使用repo命令

```bash
export REPO=$(mktemp /tmp/repo.XXXXXXXXX)
curl -o ${REPO} https://storage.googleapis.com/git-repo-downloads/repo
gpg --recv-key 8BB9AD793E8E6153AF0F9A4416530D5E920F5C65
curl -s https://storage.googleapis.com/git-repo-downloads/repo.asc | gpg --verify - ${REPO} && install -m 755 ${REPO} /usr/local/bin/repo
```

# 获取源码

参考[building QEMU v8](https://optee.readthedocs.io/en/latest/building/devices/qemu.html#qemu-v8)即可

## 编译toolchain

- 其中编译toolchain时注意使用以下命令，否则报错：

	> ccache: error: execv of /home/optee/build/../toolchains/aarch32/bin/arm-linux-gnueabihf-gcc failed: No such file or directory

```bash
make -f toolchain.mk toolchains -j2
```

​	`make toolchains`命令没有下载aarch32的编译链接工具

## 编译运行

建议使用多线程`sudo make run -j`提高编译速度

### 报错1

> uuid/uuid.h: No such file or directory

解决：`aptitude install uuid-dev`
​	
如果用的是国内源还需要

- `unset http(s) proxy`

- 或者`export no_proxy="mirrors.tuna.tsinghua.edu.cn,mirrors.aliyun.com"`

安装时有可能会提示一个依赖包冲突，选择回退这个依赖包版本，并继续安装`uuid-dev`

### 报错2

> Your PATH contains spaces, TABs, and/or newline (\n) characters.
> This doesn't work. Fix you PATH.

可是路径中并没有空格，最后参考[这篇](https://blog.csdn.net/weixin_45502929/article/details/119642271)，`sudo make run`（可是我用的就是root用户身份啊）

### 报错3

> ERROR: pkg-config binary 'pkg-config' not found

安装`apt-get install pkg-config`

### 报错4

> ERROR: glib-2.56 gthread-2.0 is required to compile QEMU

应该是qemu的依赖不全，去[Hosts/Linux - QEMU](https://wiki.qemu.org/Hosts/Linux)查看需要安装哪些包：

`apt-get install git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev ninja-build`

这些包之前环境配置时都安装过，可能是当时没安装全。

安装过程中可能需要downgrade一些包。

# 测试是否成功运行

最后等了一会儿终于运行成功，弹出两个terminal，一个是Normal World(NW)，一个是Secure World(SW)。

发现这两个terminal都是copy mode，需要在qemu终端（即原终端）输入`c`，然后NW显示`buildroot login`

> - qemu输入`help`，可以看到命令`cont|c`，是用来resume emulation
> - [buildroot](https://buildroot.org/)可以通过交叉编译生成嵌入式Linux系统. 当然x86_64也能用

输入root后，可以在NW运行:

- `xtest`

  - 测试使用的CA，会调用TA中的各种功能，包括检查基本算法接口、安全存储接口等。

  ![image-20221016121254203](https://xdo0.github.io/imgsrc/image-20221016121254203.png)

- `optee_example_hello_world`

  - 示例CA，执行一些简单的打印操作

  ![image-20221016121438481](https://xdo0.github.io/imgsrc/image-20221016121438481.png)

---

这样optee就彻底配置成功了，下次使用直接`sudo make run -j`就可以了，启动速度也快不少。
