---
layout: post
title: "在Ubuntu 20.04上安装Sourceinsight4"
date:   2024-09-13
tags: [web]
comments: true
author: weijun23
---

下载：Source_Insight.rar 链接: https://pan.baidu.com/s/1N6nR6C1A1U1u-uEkfwHGVw 提取码: 91cw

## 1）如果之前安装过sourceinsight，需清理安装环境，防止之前有残留文件 (因路径中有空格，所以路径中有转义字符)
```shell
rm -rf /root/.wine/drive_c/Program\ Files\ /Source\ Insight\ 4.0
rm -rf /root/.wine/drive_c/ProgramData/Source\ Insight
```
## 2）安装sourceinsight4
```shell
unrar x Source_Insight.rar
wine sourceinsight40121-setup.exe
```
## 3）修改sourceinsight4.exe文件

用16进制打开exe文件：
```shell
root@linux:~/.wine/drive_c/Program Files (x86)/Source Insight 4.0# hexedit sourceinsight4.exe
```
查找c800 0000 742a，74修改成eb，注意，有2处需要修改。

## 4）生成license文件
```shell
unrar si4_kgen_unis.rar
wine si4_kgen_unis.exe
```
ActID、Serial可以随便点几次，然后点击Generate生产lic文件，并保存下来。

## 5）打开sourceinsight，导入刚刚生成的lic文件
```shell
wine ~/.wine/drive_c/Program\ Files\ /Source\ Insight\ 4.0/sourceinsight4.exe
```
（由于wine自身问题，第一次启动sourceinsight，点击菜单栏可能没有反应，把sourceinsight最小化，再最大化，就正常了）

## 6）禁止sourceinsight联网检查license

sourceinsight会频繁向服务器发送信息，检查license的合法性，通过抓包，发现license服务器ip及域名如下：sls.sourceinsight.com 所以可执行命令 
```shell
iptables -t filter -w -I OUTPUT -p tcp -d 54.186.228.3 -j DROP
```
把送往license服务器的报文禁掉即可。

## 7）开机启动规则

```shell
iptables-save > /etc/iptables.rules

vi /etc/network/if-pre-up.d/ip_rules.sh

#!/bin/bash
 iptables-restore < /etc/iptables.rules
chmod 755 /etc/network/if-pre-up.d/ip_rules.sh
```

## 8）中文

上传win10里面的字体文件（windows/fonts/）到~/.wine/drive_c/windows/fonts/
```shell
cp Dengb.ttf  Dengl.ttf  Deng.ttf  msyhbd.ttc  msyhl.ttc  msyh.ttc  simfang.ttf  simkai.ttf  SIMLI.TTF  simsun.ttc ~/.wine/drive_c/windows/Fonts`
```
用win10的riched20.dll替换：
```bash
cd ~/.wine/drive_c/windows/system32
mv riched20.dll riched20.dll_bak
cp ~/Downloads/riched20.dll ./
```

编辑注册表：

```bash
wine regedit
[HKEY_LOCAL_MACHINE\\Software\\Microsoft\\WindowsNT\\CurrentVersion\\FontSubstitutes]
MS Shell Dlg = SimSun
MS Shell Dlg 2 = SimSun
Tahoma = SimSun (手动新建字符串)
```

## 9）窗口默认字体大小设置 fonts

窗口字体大小与样式设置步骤：

Preferences->Colors&Fonts->Set Panel Fonts and Colors。 这里设置只对上面

窗口1：符号窗口（Symbol Window）和窗口4：项目文件夹浏览窗口（Project Folder Browser）有效，另外两个窗口无效。

窗口2：上下文窗口（Context Window）字体大小设置如下： 在面板内右击->Context Window Options->scaling。

窗口3：引用关系窗口（Relation Window）字体大小设置如下： 窗口内右击->Relation Window Options->Font

## 10） 增加字体

在win10中找到字体：C:\Windows\Fonts Courier New
有四个文件：cp courbd.ttf courbi.ttf couri.ttf cour.ttf ~/.wine/drive_c/windows/Fonts
重新打开souceinsight可以选择.
