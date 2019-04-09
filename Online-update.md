# nxp online update

### Host 配置

- 安装如下包

```
# apt-get install xinetd tftpd tftp
```

- 重启 xinetd service:

```
# /etc/init.d/xinetd restart
```

- 测试tftp是否正常

拷贝一个文件(test.txt)到 ** /srv/tftp ** 目录下，然后：

```
$ tftp 127.0.0.1
tftp> get test.txt
```

如果正常获取到测试文件，Host 配置成功。

### 制作 rootfs ubifs image

- 在编译nxp的ubuntu根目录下创建一个目录：

```
$ sudo mkdir /tftpboot
$ sudo chmod -R 777 /tftpboot
$ sudo chown -R root /tftpboot
```

- 解压 droi-fsl-image-imx-imx6ull14x14evk.tar.bz2 到 /tftpboot

tftpboot内容如下：

```
$ cd /tftpboot
$ ls -l
total 60
drwxr-xr-x  2 root root 4096 Apr  2 19:19 bin
drwxr-xr-x  2 root root 4096 Mar 15 11:55 boot
drwxr-xr-x  2 root root 4096 Mar 15 11:55 dev
drwxr-xr-x 39 root root 4096 Apr  2 19:19 etc
drwxr-xr-x  3 root root 4096 Apr  2 19:19 home
drwxr-xr-x  7 root root 4096 Mar 15 12:18 lib
drwxr-xr-x  2 root root 4096 Mar 15 11:55 media
drwxr-xr-x  2 root root 4096 Mar 15 11:55 mnt
drwxr-xr-x  2 root root 4096 Mar 15 11:55 proc
drwxr-xr-x  2 root root 4096 Mar 15 11:55 run
drwxr-xr-x  2 root root 4096 Apr  2 19:19 sbin
drwxr-xr-x  2 root root 4096 Mar 15 11:55 sys
drwxrwxrwt  2 root root 4096 Mar 15 11:55 tmp
drwxr-xr-x 10 root root 4096 Mar 15 12:32 usr
drwxr-xr-x  7 root root 4096 Mar 15 12:36 var
```

- 安装mtd-utils

```
$ sudo apt-get install mtd-utils
```

- 生成ubi image

```
$ sudo mkfs.ubifs -F -q -r /tftpboot -m 2048 -e 126976 -c 1120 -o ~/ubifs.img
```

添加一个配置文件 ubinize.cfg, 内容如下：
```
[ubifs]
mode=ubi
image=/home/zwb/ubifs.img
vol_id=0
vol_size=140MiB
vol_type=dynamic
vol_name=rootfs
vol_flags=autoresize
```

其中 image 路径为mkfs.ubifs生成的img文件，路径要对应

最后用ubinize命令生成最终的ubifs img:

```
$ sudo ubinize -o ~/ubi.img -m 2048 -p 128KiB ~/ubinize.cfg
```

ubi.img 即为用来刷机的镜像文件

### 升级

- 拷贝 ** ubi.img ** 到 Host 目录 ** /srv/tftp **

- nxp 预置脚本 /etc/init.d/update.sh, 传入mac 地址， nxp ip地址，Host ip:

```
$ sh /etc/init.d/update.sh 00:11:22:33:44:55 192.168.100.20 192.168.100.200
```
执行 update.sh 后，自动重启进入uboot, uboot 自动刷入 ubi.img 文件。


### nxp nand 分区说明

- nand 分区查看：
```
# cat /proc/mtd
dev:    size   erasesize  name
mtd0: 04000000 00020000 "boot"
mtd1: 01000000 00020000 "kernel"
mtd2: 01000000 00020000 "dtb"
mtd3: 00100000 00020000 "misc"
mtd4: 09f00000 00020000 "rootfs"
```

- 分区起始地址：
由上面的分区表计算出分区地址：

```
0x00000000 -- 0x04000000  boot
0x04000000 -- 0x05000000  kernel
0x05000000 -- 0x06000000  dtb
0x06000000 -- 0x06100000  misc
0x06100000 -- 0x10000000  rootfs
```

### mkfs.ubifs 参数说明

查看 mtd4 (rootfs) info:

```
root@imx6ull14x14evk:~# mtdinfo /dev/mtd4 -u
mtd4
Name:                           rootfs
Type:                           nand
Eraseblock size:                131072 bytes, 128.0 KiB
Amount of eraseblocks:          1272 (166723584 bytes, 159.0 MiB)
Minimum input/output unit size: 2048 bytes
Sub-page size:                  2048 bytes
OOB size:                       64 bytes
Character device major/minor:   90:8
Bad blocks are allowed:         true
Device is writable:             true
Default UBI VID header offset:  2048
Default UBI data offset:        4096
Default UBI LEB size:           126976 bytes, 124.0 KiB
Maximum UBI volumes count:      128
```

-m   Minimum input/output unit size: 2048 bytes
-e   Default UBI LEB size:           126976 bytes, 124.0 KiB
-c   1120 = (vol_size 140MiB * 1024 ) / 128





