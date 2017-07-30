---
title: VS2013编译配置openssl
date: 2017-05-14
categories: Compile & Installation
tags: Sofeware
---

&emsp;&emsp;简单介绍如何使用VS2013编译配置openssl，并添加到工程中
<!--more-->

### VS2013编译openssl1.1.0e

1.下载OpenSSL http://www.openssl.org ，并解压到指定目录，例如 D:\openssl

2.下载并安装Perl  http://www.activestate.com/ActivePerl

3.新建一个存放openssl库文件的目录，例如 D:\openssl\openssl_lib

4.打开VS2013控制台命令提示(root_dir为VS安装目录)
Visual Studio 2013, 32-bit: Open CMD.EXE and run 
```
root_dir\Microsoft Visual Studio 12.0\VC\bin\vcvars32.bat  
```
Visual Studio 2013, 64-bit: Open CMD.EXE and run 
```
root_dir\Microsoft Visual Studio 12.0\VC\bin\amd64\vcvars64.bat
```

5.进入openssl源代码目录路径 
```
cd D:\openssl
```

6.依次执行以下命令
```
perl Configure VC-WIN32 <no-asm> --perfix=D:\openssl\openssl_lib //<no-asm>为可选参数，忽略汇编处理

nmake

nmake test

nmake install
```

7.编译成功后，会在openssl_lib目录下生成3个文件目录

bin: 主要有libcrypto-1_1.dll、libssl-1_1.dll动态链接库文件和openssl.exe文件
     
lib: 静态链接库libcrypto.lib、libssl.lib文件

include\openssl: 相关头文件


### VS2013中使用openssl

1.新建VS工程，将openssl库文件(.lib、.dll)和头文件拷至工程的相应目录中

2.配置路径

静态库: 链接附加库目录至libssl.lib、libcrypto.lib库目录，并在附加依赖项下添加.lib文件

动态库：libcrypto-1_1.dll、libssl-1_1.dll放置到.exe输出目录中（Debug/Release)

头文件：链接附加包含目录至 include 目录


*关于动态库dll的链接问题：
（1）包含exe文件的输出目录
（2）进程的工作目录（工程主目录）
（3）Windows系统目录
（4）Windows目录
（5）列在Path环境变量中的一系列目录
  在VS工程中设置Path路径：配置属性>调试>环境 PATH=dll所在路径，可以是相对路径（不要有空格，最后已半角分号结束）

*注：openssl的1.1.0及以上版本和1.0.2及以下版本的编译方式有所不同

### reference
- http://p-nand-q.com/programming/windows/building_openssl_with_visual_studio_2013.html
- http://blog.csdn.net/u010725842/article/details/50295235




