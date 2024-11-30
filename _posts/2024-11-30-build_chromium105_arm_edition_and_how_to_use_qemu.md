---
title: 编译chromium105 arm版，以及怎么使用qemu
date: 2024-11-30 10:56:51 +0800
categories: [Blogging]
tags: [chromium]
---


编译chromium105 arm版，以及怎么使用qemu

## 1、获取代码

```bash
git clone --depth 1 --branch 105.0.5195.148 https://chromium.googlesource.com/chromium/src
gclient sync --nohooks --no-history -v -j1
gclient runhooks -v -j1 
```

## 2、args.gn和编译

新建编译参数文件args.gn
```python
target_os = "linux"
target_cpu = "arm"
is_component_build = false
is_debug = true
symbol_level=2
blink_symbol_level=0
v8_symbol_level=0

enable_nacl = false
```
一切就绪，开始编译
```bash
cd src
mkdir out/
mv ../args.gn out/args.gn
gn gen out/
autoninja -C out/ content_shell
```

## 3、Qemu

现成的，预构建的 QEMU Debian 镜像：https://people.debian.org/~aurel32/qemu/armhf/
```bash
qemu-system-arm -M vexpress-a9 -kernel vmlinuz-3.2.0-4-vexpress -initrd initrd.img-3.2.0-4-vexpress -drive if=sd,file=debian_wheezy_armhf_standard.qcow2 -append "root=/dev/mmcblk0p2"
```

自己编译镜像
```bash
wget https://mirror.bjtu.edu.cn/kernel/linux/kernel/v5.x/linux-5.10.tar.xz
tar -xvf linux-5.10.tar.xz
make CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm vexpress_defconfig
make CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm
生成的内核镱像位于arch/arm/boot/zImage, 设备树 arch/arm/boot/dts/vexpress-v2p-ca9.dtb

wget https://ftp.denx.de/pub/u-boot/u-boot-2020.10.tar.bz2
tar -xvf u-boot-2020.10.tar.bz
ls configs/ | grep vexpress
make vexpress_ca9x4_defconfig
make CROSS_COMPILE=arm-linux-gnueabihf- all
最终编译生成 elf 格式的可执行文件 u-boot 和纯二进制文件u-boot.bin，其中 QEMU 可以启动的为 elf 格式的可执行文件 u-boot

wget https://git.busybox.net/busybox/snapshot/busybox-1_33_1.tar.bz2
tar xjvf  busybox-1_33_1.tar.bz2
cd busybox-1_33_1
CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm make menuconfig
     Setting->[*] Build static binary (no sharded libs)
CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm make -j8
CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm make install -j8
安装（在busybox-1_33_1/_install目录下安装）
```

宿主服务器网络配置
```bash
wuwj:~/qemu$ sudo ip tuntap add dev tap0 mode tap
wuwj:~/qemu$ sudo ip link set dev tap0 up
wuwj:~/qemu$ sudo ip address add dev tap0 192.168.2.128/24
wuwj:~/qemu$ sudo iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -o wlan0 -j MASQUERADE
这条规则的意思是，来自 192.168.2.0/24，且从 wlan0 出去的包，要进行 NAT，同时会对返回的包进行 NAT。如果只有一个子网， -s 192.168.2.0/24 可以省略。

```

生成rootfs镜像
```bash
sudo rm -rf rootfs
mkdir -p rootfs/{dev,etc/init.d,lib,proc,tmp,sys,media,mnt}
sudo cp busybox-1_33_1/_install/* -r rootfs/
sudo cp -P /usr/arm-linux-gnueabi/lib/* rootfs/lib/
echo "root:x:0:0:root:/root:/bin/sh" > rootfs/etc/passwd
echo "root:x:0:" > rootfs/etc/group
echo "nameserver 8.8.8.8" > rootfs/etc/resolv.conf
(
cat <<EOF
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t tmpfs mdev /dev
mkdir -p /dev/pts
mount -t devpts devpts /dev/pts
mount -o noexec -t tmpfs tmpfs /tmp
mount -t tmpfs tmpfs /media
mknod /dev/console c 5 1
mknod /dev/ptmx c 5 2
mknod /dev/null c 1 3
mknod /dev/tty1 c 4 1
mknod /dev/tty2 c 4 2
mknod /dev/tty3 c 4 3
mknod /dev/tty4 c 4 4
ifconfig lo up
ifconfig eth0 up
ip addr add 192.168.2.129/24 dev eth0
route add default gw 192.168.2.128 dev eth0
EOF
) > rootfs/etc/init.d/rcS
sudo chmod +x rootfs/etc/init.d/rcS
dd if=/dev/zero of=a9rootfs.ext4 bs=1M count=32
mkfs.ext4 a9rootfs.ext4
sudo mkdir -p tmpfs
sudo mount -t ext4 a9rootfs.ext4 tmpfs/ -o loop
sudo cp -r rootfs/*  tmpfs/
sudo umount tmpfs
sudo rm -rf tmpfs
```
运行
```bash
qemu-system-arm -M vexpress-a9 -m 512M -nographic \
   -kernel linux-5.10/arch/arm/boot/zImage \
   -dtb linux-5.10/arch/arm/boot/dts/vexpress-v2p-ca9.dtb \
   -append "root=/dev/mmcblk0 rw console=ttyAMA0" -sd a9rootfs.ext4 \
   -net nic -net tap,ifname=tap0,script=no,downscript=no
```
 
qemu-system-arm                          #qemu主要配置  
-M vexpress-a9                           #模拟vexpress-a9单板  
-m 512M                                  #内存配置  
-kernel linux-5.10/arch/arm/boot/zImage  #内核路径  
-dtb linux-5.10/arch/arm/boot/dts/vexpress-v2p-ca9.dtb  #设备树路径  
-nographic                               #不使用图形化界面，只使用串口  
-append "root=/dev/mmcblk0 rw  console=ttyAMA0"  #内核启动参数(vexpress单板运行)  
-sd a9rootfs.ext4                        #SD卡印像  
-netdev user,id=net0,hostfwd=tcp::2222-:22： 创建一个用户模式网络设备，hostfwd 选项将主机的 2222 端口转发到虚拟机的 22 端口（SSH）。  
-device virtio-net-device,netdev=net0：        添加一个 VirtIO 网络设备，连接到之前创建的网络设备。  

Ctrl+A 后按 x 键 退出  
