# droi-imx

### 编译环境
- ubuntu 14.04
- 安装如下工具包

```
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat
libsdl1.2-dev xterm g++ libstdc++6 lib32stdc++6 libpulse-dev libevent-dev rpm2cpio
```
```
$ sudo apt-get install ninja-build
```

### 编译命令

repo 工程

```
$ repo init --no-repo-verify -u git@github.com:fukehan/droi-imx.git -m imx-4.1.15-2.1.0.xml
$ repo sync
```

然后：

```
$ DISTRO=fsl-imx-fb MACHINE=imx6ull14x14evk source ./droi-fsl-setup-release.sh -b build-fb
```
中间会出现license提示，一路按**空格键**，直到最后按 **y** 确认，最后创建 **build-fb** 目录，并自动cd到
该目录下，然后继续执行：

```
$ bitbake droi-fsl-image-imx
```

### 编译u-boot

clone [u-boot-imx](https://github.com/fukehan/u-boot-imx)

**master** 分支包含我们修改的 commit, **imx_v2016.03_4.1.15_2.0.0_ga** 分支为 nxp release 分支

进入 u-boot-imx 根目录，然后按照如下方法编译：
```
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabi-
$ make mx6ull_14x14_evk_nand_defconfig
$ make
```
编译完成后生成 **u-boot.imx** 文件

### 编译kernel

clone [linux-imx](https://github.com/fukehan/linux-imx)

**master** 分支包含我们修改的 commit, **imx_4.1.15_2.0.0_ga** 分支为 nxp release 分支

进入根目录linux-imx :

```
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabi-
```
编译设备树：

```
$ make imx_v7_defconfig
$ make make imx6ull-14x14-evk-gpmi-weim.dtb
```

编译zImage:
```
$ make zImage
```

