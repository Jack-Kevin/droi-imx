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
$ DISTRO=fsl-imx-fb MACHINE=imx6ull14x14evk source droi-fsl-setup-release.sh -b build-fb
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

### 下载到nxp target
nxp target boot 需要4个文件： uboot, kernel-dtb, kernel-zImage, rootfs.

uboot: u-boot-imx/u-boot.imx

rootfs: build-fb/tmp/deploy/images/imx6ull14x14evk/droi-fsl-image-imx-imx6ull14x14evk.tar.bz2

zImage:  arch/arm/boot/zImage

dtb:  arch/arm/boot/dts/imx6ull-14x14-evk-gpmi-weim.dtb

使用[MFG-tool](https://github.com/fukehan/mfg-tool)下载到target(mfg tool 只支持windows,以下操作是在windows上进行)：

- 拷贝上述4个文件到mfg tool 目录下 "mfgtools\Profiles\Linux\OS Firmware\imx6ull14x14evk"
- target 通过 usb cable 和 windows 连接:
  ![usb connetc](https://github.com/fukehan/droi-imx/blob/master/img/usb-download-connet.jpg)
- 点击mfgtool2-yocto-mx-evk-nand.vbs，弹出下载框：
  ![下载框](https://github.com/fukehan/droi-imx/blob/master/img/nxp-download.png)
  
- 接通12V电源，点击start 开始下载

如果target 已经下载过， 需要先擦除nand才能继续下载：

- 用 uart 连接target和电脑，target 上电开机后， u-boot有3s的等待时间，按任意键进入u-boot commnd界面， 
  运行命令：nand erase.chip clean
- 然后正常进行下载

uart 连接方式：
![uart](https://github.com/fukehan/droi-imx/blob/master/img/uart-connet.jpg)

第1根 TX， 第2根 RX， 第3根是地线

linux 下烧录（以下在ubuntu 14.04 LTS 上测试成功）:

- 解压[MFG-tool](https://github.com/fukehan/mfg-tool)到ubuntu
- 修改 mfgtool2-yocto-mx-evk-nand.vbs, 其中""NAND Flash"" 改为""NAND""
- 修改 ucl2.xml, 找到:
  ```
  <LIST name="NAND Flash" desc="Choose NAND as media">
  ```
  修改为：
  ```
  <LIST name="NAND" desc="Choose NAND as media">
  ```
  此处name 保持和mfgtool2-yocto-mx-evk-nand.vbs中的一致。
  
  另外，ucl2.xml文件开头删除多余的，只保留：
  ```
  <CFG>
    <STATE name="BootStrap" dev="MX6UL" vid="15A2" pid="0080"/>
    <STATE name="Updater"   dev="MSC" vid="066F" pid="37FF"/>
  </CFG>
  ```
  
- 修改linux-runvbs.sh，mfgtoolcli权限
  ```
  $ chmod a+x linux-runvbs.sh
  $ chmod a+x mfgtoolcli
  ```
- 执行下载脚本
  ```
  $ sudo ./linux-runvbs.sh mfgtool2-yocto-mx-evk-nand.vbs
  ```
  如果提示如下错误：
  ```
  init op Failed code# 24
  ```
  需要检查是否识别到usb devices, lsusb 看下是否找到：
  ```
  Bus 003 Device 056: ID 15a2:0080 Freescale Semiconductor, Inc.
  ```
  如果没有识别usb, 请重新上电一次。USB连接正确，提示如下界面：
  ![usb](https://github.com/fukehan/droi-imx/blob/master/img/Linux-usb-succsess.png)
  
  然后在重新上电一起，即可开始下载：
  ![usb](https://github.com/fukehan/droi-imx/blob/master/img/linux-usb-down.png)
  
- 如果nand不是第一次下载，同样需要清空nand, 连接uart, 进入u-boot清除nand, 如下在ubuntu上使用putty连接uart:
  putty如下配置：
  <div align=center><img src="https://github.com/fukehan/droi-imx/blob/master/img/linux-putty-config.png"/></div>
  
  上电后，按任意键进入u-boot command:
  <div align=center><img src="https://github.com/fukehan/droi-imx/blob/master/img/linux-uboot-nand.png"/></div>
  
  输入"nand erase.chip clean" 清空nand, 按上面的步骤下载。
  





