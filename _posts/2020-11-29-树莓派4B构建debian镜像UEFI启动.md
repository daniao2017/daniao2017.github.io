---
layout:     post
title:      树莓派4B构建debian镜像UEFI启动 
date:       2020-11-29
author:     肥仔
catalog: true
tags:
    - 树莓派
    - UEFI
    - debian镜像
--- 

# 树莓派4B构建debian镜像UEFI启动
## 前言
今天按照大佬的博客[树莓派俱乐部官方 Debian 系统镜像 支持UEFI](https://raspberrypi.club/341.html)跑了遍,完整的UEFI镜像构建过程。

包括镜像分区/挂载，根文件系统的制作，内核的移植，UEFI固件的烧录。


正常的逻辑是openEuler系统能被其他的系统，通过UEFI启动，如u盘启动。或者是本机运行双系统或者是本身支持引导其他系统，相关知识待补充。印象中，原生的openEuler是支持UEFI启动的。

## 制作空白镜像

制作1500MB的空白镜像，镜像名称叫debian.img

>dd if=/dev/zero of=debian.img bs=1M count=1500 status=progress

创建分区表

>parted debian.img mktable msdos

创建分区，分区作用如下，也可以只用两个分区

- 第一分区（256MB）：EFI引导分区
- 第二分区（64MB）：存放内核
- 第三分区（剩余空间）：根目录

使用fdisk分对应大小的分区

>fdisk debian.img

```
n
p
1
2048
526336
n
p
2
526337
657409
n
p
3
657410
3071999
w
```

**第一分区是256MB所以526336-2048=256x2048，第二分区64MB所以657409-526337=64x2048依此类推**

### 安装使用kpartx映射分区

>sudo apt-get install kpartx

>kpartx -av debian.img

分区映射在/dev/mapper/下,接下来创建文件系统

第一分区是fat32，第二分区是ext4，第三分区时f2fs。使用时，可能会提示无`f2fs`,按照即可。

```
mkfs.vfat -F 32 /dev/mapper/loop0p1

mkfs.ext4 -L KERNEL /dev/mapper/loop0p2

mkfs.f2fs -l ROOTFS /dev/mapper/loop0p3
```

取消映射
> kpartx -d debian.img

要使树莓派能够启动，还需要添加lba的标识

>parted -s debian.img -- toggle 1 lba

在某些开放板上，还需要添加boot的标识，当然，这在树莓派上没必要。

>parted -s debian.img -- toggle 1 boot

到此，一个空白镜像已经完成了。

## 构建根目录

安装所需软件包

>sudo apt install debootstrap debian-keyring qemu-user-static

使用debootstrap创建根目录

```
mkdir rootfs

sudo debootstrap --arch=arm64 --foreign --no-check-gpg buster ./rootfs http://mirrors.tuna.tsinghua.edu.cn/debian

cp /usr/bin/qemu-aarch64-static rootfs/usr/bin/

cd rootfs

LC_ALL=C LANGUAGE=C LANG=C chroot . /debootstrap/debootstrap --second-stage

LC_ALL=C LANGUAGE=C LANG=C chroot . dpkg --configure -a
```

安装自己所需的软件包
```
LC_ALL=C LANGUAGE=C LANG=C chroot rootfs apt-get install -y sudo ssh net-tools ethtool wireless-tools network-manager iputils-ping rsyslog alsa-utils bash-completion gnupg busybox kmod --no-install-recommends
```

为此系统添加用户，假设系统用户名为pi

```
chroot rootfs adduser pi && addgroup pi adm && addgroup pi sudo && addgroup pi audio
```
然后需要输入一些信息，请务必记住**输入的密码**

开启对于armhf的兼容（可选）
```
chroot rootfs dpkg --add-architecture armhf
chroot rootfs apt-get install libc6:armhf
```

开启armel的兼容
```
chroot rootfs dpkg --add-architecture armel
chroot rootfs apt-get install libc6:armel
```

### 开始配置根目录

添加hosts

>echo '127.0.0.1	raspberrypi' >> rootfs/etc/hosts

编辑主机名为raspberrypi

```
cat /dev/null > rootfs/etc/hostname

echo 'raspberrypi' >> rootfs/etc/hostname
```

编辑fstab挂载信息

```
cat <<EOF >> rootfs/etc/fstab

proc            /proc           proc    defaults          0       0

/dev/mmcblk0p2  /boot           ext4    defaults          0       0

/dev/mmcblk0p1  /boot/efi       vfat    defaults          0       2

/dev/mmcblk0p3  /               f2fs    defaults,noatime  0       1

# a swapfile is not a swap partition, no line here

#   use  dphys-swapfile swap[on|off]  for that

tmpfs /tmp tmpfs defaults,noatime,nosuid,size=100m 0 0

tmpfs /var/tmp tmpfs defaults,noatime,# 树莓派4B构建debian镜像UEFI启动
## 前言
今天按照大佬的博客[树莓派俱乐部官方 Debian 系统镜像 支持UEFI](https://raspberrypi.club/341.html)跑了遍,完整的UEFI镜像构建过程。

包括镜像分区/挂载，根文件系统的制作，内核的移植，UEFI固件的烧录。


正常的逻辑是openEuler系统能被其他的系统，通过UEFI启动，如u盘启动。或者是本机运行双系统或者是本身支持引导其他系统，相关知识待补充。印象中，原生的openEuler是支持UEFI启动的。

## 制作空白镜像

制作1500MB的空白镜像，镜像名称叫debian.img

>dd if=/dev/zero of=debian.img bs=1M count=1500 status=progress

创建分区表

>parted debian.img mktable msdos

创建分区，分区作用如下，也可以只用两个分区

- 第一分区（256MB）：EFI引导分区
- 第二分区（64MB）：存放内核
- 第三分区（剩余空间）：根目录

使用fdisk分对应大小的分区

>fdisk debian.img

```
n
p
1
2048
526336
n
p
2
526337
657409
n
p
3
657410
3071999
w
```

**第一分区是256MB所以526336-2048=256x2048，第二分区64MB所以657409-526337=64x2048依此类推**

### 安装使用kpartx映射分区

>sudo apt-get install kpartx

>kpartx -av debian.img

分区映射在/dev/mapper/下,接下来创建文件系统

第一分区是fat32，第二分区是ext4，第三分区时f2fs。使用时，可能会提示无`f2fs`,按照即可。

```
mkfs.vfat -F 32 /dev/mapper/loop0p1

mkfs.ext4 -L KERNEL /dev/mapper/loop0p2

mkfs.f2fs -l ROOTFS /dev/mapper/loop0p3
```

取消映射
> kpartx -d debian.img

要使树莓派能够启动，还需要添加lba的标识

>parted -s debian.img -- toggle 1 lba

在某些开放板上，还需要添加boot的标识，当然，这在树莓派上没必要。

>parted -s debian.img -- toggle 1 boot

到此，一个空白镜像已经完成了。

## 构建根目录

安装所需软件包

>sudo apt install debootstrap debian-keyring qemu-user-static

使用debootstrap创建根目录

```
mkdir rootfs

sudo debootstrap --arch=arm64 --foreign --no-check-gpg buster ./rootfs http://mirrors.tuna.tsinghua.edu.cn/debian

cp /usr/bin/qemu-aarch64-static rootfs/usr/bin/

cd rootfs

LC_ALL=C LANGUAGE=C LANG=C chroot . /debootstrap/debootstrap --second-stage

LC_ALL=C LANGUAGE=C LANG=C chroot . dpkg --configure -a
```

安装自己所需的软件包
```
LC_ALL=C LANGUAGE=C LANG=C chroot rootfs apt-get install -y sudo ssh net-tools ethtool wireless-tools network-manager iputils-ping rsyslog alsa-utils bash-completion gnupg busybox kmod --no-install-recommends
```

为此系统添加用户，假设系统用户名为pi

```
chroot rootfs adduser pi && addgroup pi adm && addgroup pi sudo && addgroup pi audio
```
然后需要输入一些信息，请务必记住**输入的密码**

开启对于armhf的兼容（可选）
```
chroot rootfs dpkg --add-architecture armhf
chroot rootfs apt-get install libc6:armhf
```

开启armel的兼容
```
chroot rootfs dpkg --add-architecture armel
chroot rootfs apt-get install libc6:armel
```

### 开始配置根目录

添加hosts

>echo '127.0.0.1	raspberrypi' >> rootfs/etc/hosts

编辑主机名为raspberrypi

```
cat /dev/null > rootfs/etc/hostname

echo 'raspberrypi' >> rootfs/etc/hostname
```

编辑fstab挂载信息

```
cat <<EOF >> rootfs/etc/fstab

proc            /proc           proc    defaults          0       0

/dev/mmcblk0p2  /boot           ext4    defaults          0       0

/dev/mmcblk0p1  /boot/efi       vfat    defaults          0       2

/dev/mmcblk0p3  /               f2fs    defaults,noatime  0       1

# a swapfile is not a swap partition, no line here

#   use  dphys-swapfile swap[on|off]  for that

tmpfs /tmp tmpfs defaults,noatime,nosuid,size=100m 0 0

tmpfs /var/tmp tmpfs defaults,noatime,nosuid,size=30m 0 0

tmpfs /var/log tmpfs defaults,noatime,nosuid,mode=0755,size=100m 0 0

tmpfs /var/run tmpfs defaults,noatime,nosuid,mode=0755,size=2m 0 0

tmpfs /var/spool/mqueue tmpfs defaults,noatime,nosuid,mode=0700,gid=12,size=30m 0 0

EOF
```

下载一些设备的firmware

```
cd root/lib

git clone https://github.com/rpi-distro/firmware-nonfree

mv firmware-nonfree firmware

rm -rf firmware/.git
```

nosuid,size=30m 0 0

tmpfs /var/log tmpfs defaults,noatime,nosuid,mode=0755,size=100m 0 0

tmpfs /var/run tmpfs defaults,noatime,nosuid,mode=0755,size=2m 0 0

tmpfs /var/spool/mqueue tmpfs defaults,noatime,nosuid,mode=0700,gid=12,size=30m 0 0

EOF
```

下载一些设备的firmware

```
cd root/lib

git clone https://github.com/rpi-distro/firmware-nonfree

mv firmware-nonfree firmware

rm -rf firmware/.git
```

