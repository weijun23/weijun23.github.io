---
title: 在ubuntu 20.04上编译老版本chromium 53.0.2785.134
date: 2024-11-25 10:56:51 +0800
categories: [Blogging]
tags: [chromium]
---

在ubuntu 20.04上编译老版本chromium 53.0.2785.134

## 1、获取depot_tools

下载最新depot_tools,，这个压缩包里带.git目录，下载后解压(注：git回退到和chromium同时期的版本，防止运行gclient意外)
```bash
wget https://storage.googleapis.com/chrome-infra/depot_tools.zip
unzip depot_tools.zip  -d ./depot_tools
```
最后将depot_tools添加到环境变量PATH中。
```bash
export PATH=`pwd`/depot_tools:$PATH
```

## 2、下载chromium代码

```bash
git clone --depth 1 --branch 53.0.2785.134 https://gitee.com/mirrors/chromium.git
mv chromium src
```

## 3、获取第三方源代码 (这步只能在Ubuntu16.04上运行)

手工创建gclient配置文件.gclient
```python
solutions = [
  {
    "name": "src", # 拉去后存放的目录,这是chromium的存放目录，其他三方代码还是放到src
    "url": "https://gitee.com/mirrors/chromium.git", # 要拉取的仓库地址，solution地址
    "custom_deps": {}, # 这是一个可选的字典对象，会覆盖工程的"DEPS"文件定义的条目
    "deps_file": "",   # deps 文件名（如果找不到，会找DEPS?）
    "managed": False,  # 使用 unmanaged 模式,unmanaged solution; skipping src,不拉取chromium代码，已经下载了
    "safesync_url": "",
  },
]
```
使用gclient sync --nohooks命令下载chromium的第三方源代码，比如v8源代码，他是根据DEPS文件下载，这里必须要用到git代理  
拉取代码之后不执行hooks,-v显示详细每一步  
```bash
gclient sync --nohooks --no-history -v -j1
```
利用gclient runhooks命令自动下载编译环境的文件，如libcxx，描述在DEPS里的hooks节点  
可以使用下面命令来手动执行hooks，这里用到vpython，老版本depot_tools才有，这是用老depot_tools的原因了
```bash
gclient runhooks -v -j1 
``` 
## 4、编译

新建编译参数文件args.gn，如果需要更多详细的配置参数，查看官网http://www.chromium.org/developers/gn-build-configuration
```python
target_os = "linux"
target_cpu = "x64"
is_component_build = false
is_debug = true
symbol_level=2
```
symbol_level = 2：生成完整的调试符号，适合调试。0: 不生成调试符号。1: 生成基本的调试符号，适合发布版本。  
is_component_build = false：使用静态链接构建，所有依赖项都包含在最终的可执行文件中。  

一切就绪，开始编译
```bash
cd src
mkdir out/
mv ../args.gn out/args.gn
gn gen out/
ninja -C out/ content_shell
```
附加命令：gn args --list out/，可以看配置

## 5、运行

```bash
cd src/out/; LD_LIBRARY_PATH=/home/wuwj/chromium/lib/ ./content_shell
```
因为是chromium需要的库和ubuntu20.04里带的库的新老关系，运行时需要旧库，所以要加上LD_LIBRARY_PATH，库从chromium编译里有
```bash
wuwj:~/chromium$ ll lib/
总用量 3072
lrwxrwxrwx 1 wuwj wuwj      21 1月  31  2013 libcairo.so -> libcairo.so.2.11200.2
lrwxrwxrwx 1 wuwj wuwj      21 1月  31  2013 libcairo.so.2 -> libcairo.so.2.11200.2
-rw-r----- 1 wuwj wuwj 1012888 1月  31  2013 libcairo.so.2.11200.2
-rwxrwxr-x 1 wuwj wuwj 1443856 9月  27 20:59 libfreetype.so.6*
lrwxrwxrwx 1 wuwj wuwj      19 3月  12  2015 libgcrypt.so -> libgcrypt.so.11.7.0
lrwxrwxrwx 1 wuwj wuwj      19 3月  12  2015 libgcrypt.so.11 -> libgcrypt.so.11.7.0
-rw-r----- 1 wuwj wuwj  520224 3月  12  2015 libgcrypt.so.11.7.0
lrwxrwxrwx 1 wuwj wuwj      18 11月 21 16:03 libpng12.so.0 -> libpng12.so.0.49.0
-rw-r----- 1 wuwj wuwj  158640 11月 21 16:03 libpng12.so.0.49.0
```

附加快照下载地址，这是google自己编译好的[content_shell](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Linux_x64/424785/)

