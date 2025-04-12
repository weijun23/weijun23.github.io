---
title: 在ARM Linux上使用ldd
date: 2024-12-03 10:56:51 +0800
categories: [Blogging]
tags: [chromium]
mermaid: true
---

**1. ldd介绍**

ldd——library dynamic dependency。用于输出可执行程序或共享库依赖的共享库列表。
```shell
ldd /bin/ls:
```
![pic](https://pic3.zhimg.com/v2-da8e62a16a02ff48b232a2d8d1a721b8_1440w.jpg)

**2. ldd实现原理**

ldd通过设置环境变量来实现输出依赖的功能。当环境变量**LD_TRACE_LOADED_OBJECTS**的值不为空时，可执行程序在被执行的时候，只会显示该程序的依赖而不会真正的被执行。
![pic](https://pic1.zhimg.com/v2-7ae1baa7c7741ddecd628a5961b196bc_1440w.jpg)

ldd命令最根本的实现原理是执行了**ld-linux.so.xx。(动态库也可以被执行？？？需要深入探究！！！)**


**3. ldd说明**

ldd是一个shell脚本，不是可执行程序
```shell
cat /usr/bin/ldd:
```
```bash
#! /bin/bash
# Copyright (C) 1996-2016 Free Software Foundation, Inc.
# This file is part of the GNU C Library.
​
# The GNU C Library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
​
# The GNU C Library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
​
# You should have received a copy of the GNU Lesser General Public
# License along with the GNU C Library; if not, see
# <http://www.gnu.org/licenses/>.
​
​
# This is the `ldd' command, which lists what shared libraries are
# used by given dynamically-linked executables.  It works by invoking the
# run-time dynamic linker as a command and setting the environment
# variable LD_TRACE_LOADED_OBJECTS to a non-empty value.
​
# We should be able to find the translation right at the beginning.
TEXTDOMAIN=libc
TEXTDOMAINDIR=/usr/share/locale
​
RTLDLIST="/lib/ld-linux.so.2 /lib64/ld-linux-x86-64.so.2 /libx32/ld-linux-x32.so.2"
warn=
bind_now=
verbose=
​
while test $# -gt 0; do
  case "$1" in
  --vers | --versi | --versio | --version)
    echo 'ldd (Ubuntu GLIBC 2.23-0ubuntu10) 2.23'
    printf $"Copyright (C) %s Free Software Foundation, Inc.
......
```

**4. ldd"移植“**

ldd的“移植”很简单：

* 将#! /bin/bash改为#! /bin/sh。arm linux平台大部分使用sh作为shell解释器；
* 修改变量RTLDLIST值。将其修改为arm linux平台下的链接动态库(ld-linux-xx.so)，多数位于/lib目录下;
