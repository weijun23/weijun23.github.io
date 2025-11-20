---
title: 在Ubuntu固定USB设备串口号
date: 2025-11-20 00:56:51 +0800
categories: [Blogging]
tags: [git]
mermaid: true
---

**步骤 1：查看设备信息**

插入USB设备后，使用以下命令查看设备的基本信息：

```bash
lsusb
```

输出示例：

```bash
Bus 002 Device 004: ID 1234:5678 My USB Device
```

记录设备的厂商ID（VID）和产品ID（PID），例如：1234 和 5678。


**步骤 2：获取设备的详细信息**

查看设备的插入日志，获取设备的串口号（如ttyUSB0）：

```bash
dmesg | grep tty
```

输出示例：

```bash
[ 1234.567890] usb 2-1: ch341-uart converter now attached to ttyUSB0
```

查看设备的属性信息：

```bash
udevadm info --name=/dev/ttyUSB0 --attribute-walk
```

替换/dev/ttyUSB0为实际设备路径。输出内容类似以下：

```bash
ATTRS{idVendor}=="1234"
ATTRS{idProduct}=="5678"
ATTRS{serial}=="A12345B67890"
```

记录idVendor、idProduct以及serial（如有）。

**步骤 3：创建udev规则**

在/etc/udev/rules.d/目录下创建新的规则文件，例如：

```bash
sudo vim /etc/udev/rules.d/99-usb-serial.rules
```

在文件中添加规则，以下为示例内容：
为设备1创建固定名称

```bash
SUBSYSTEM=="tty", ATTRS{idVendor}=="1234", ATTRS{idProduct}=="5678", ATTRS{serial}=="A12345B67890", SYMLINK+="ttyUSB_device_1"
```

为设备2创建固定名称（如有多个设备）

```bash
SUBSYSTEM=="tty", ATTRS{idVendor}=="abcd", ATTRS{idProduct}=="efgh", ATTRS{serial}=="Z98765X43210", SYMLINK+="ttyUSB_device_2"
```

替换idVendor、idProduct和serial为实际设备信息。
SYMLINK表示创建的符号链接名称（如ttyUSB_device_1）。


**步骤 4：重新加载udev规则**


保存规则文件后，重新加载udev规则：

```bash
sudo udevadm control --reload-rules
```

为确保规则生效，可以手动触发设备检测：

```bash
sudo udevadm trigger
```

**步骤 5：验证规则生效**

重新插拔设备后，检查是否生成了固定的符号链接：

```bash
ls -l /dev/ttyUSB*
```

输出示例：

```bash
lrwxrwxrwx 1 root root 7 Dec  6 12:34 /dev/ttyUSB_device_1 -> ttyUSB0
lrwxrwxrwx 1 root root 7 Dec  6 12:34 /dev/ttyUSB_device_2 -> ttyUSB1
```

确保每次插入设备后，能通过符号链接访问设备（如/dev/ttyUSB_device_1）。
注意事项
文件命名： udev规则文件的命名（如99-usb-serial.rules）与SYMLINK名称无关，但建议使用清晰的文件名便于管理。
属性匹配： idVendor和idProduct是必需的，serial属性可选（如设备有唯一序列号，建议添加）。
符号链接： SYMLINK提供固定的设备路径，便于开发和调试。
通过以上步骤，可以确保每次插入USB设备时，设备绑定到固定的串口号，提升开发效率和使用便利性。
