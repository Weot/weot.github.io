---
title: C语言开发环境配置(win)
date: 2017-09-13 07:06
updated: 2018-11-09 15:01
categories:
- 编程开发
- C语言
tags: [编程, C语言, 环境, tcc]
---
## 一. 开始前准备

> 随着win系统的不断升级, 现在已到了win10(2017年), 想要学习C语言的小伙伴发现vc6.0没办法装在win10上面, 所以就写一个教程来搭建一个简单的环境吧.


### 1. 编辑器: sublime text3

sublime text3就不用说明了, 一个简洁而又强大的编辑器

下载地址:[官网](http://www.sublimetext.com/3)

注册码: 百度搜索`"sublime tex3 注册码"`

[官方文档](http://www.sublimetext.com/docs/3/)

[非官方文档](http://docs.sublimetext.info/en/latest/index.html), 虽然非官方, 却有更多实用的东西


### 2. 编译器: tcc

`tcc`: 一个极小的C语言编译器(Tiny C Compiler), 由于win下没有自带gcc编译器, tcc是一个很好的选择, 特点:

- 非常小巧
- 速度快
- 自动编译,链接

下载地址: [tcc](https://bellard.org/tcc/)

文档地址: [文档](https://bellard.org/tcc/tcc-doc.html)


### 3. 运行方式: cmd或cmder/conemu中运行

理想运行方式: 

1. 在sublime text3中码代码
2. `ctrl+b`一键运行(中间tcc负责编译, 链接)
3. 弹出命令行窗口, 显示运行结果

cmder/conemu不是必须的选项, 只是美化效果及其他一些增强命令比较好用

下载地址: [cmder](http://cmder.net/)

附效果图:

![image](http://ozi7kdi5x.bkt.clouddn.com/8e480af026effa6.jpg)


## 二. 安装配置

### 1. 配置tcc
    
1. tcc 下载解压后放到任意文件夹, 添加tcc路径到PATH环境变量:
    
    ![image](http://ozi7kdi5x.bkt.clouddn.com/b724b67fe368e91.jpg)

2. 现在命令行窗口键入`tcc`, 有类似下面的显示, 说明配置好了

    ![image](http://ozi7kdi5x.bkt.clouddn.com/54d3d943c8efc7b.jpg)

### 2. 在sublime text3中添加tcc编译系统
    
sublime text3中可以自定义编译系统, 看起来好像十分复杂样子, 实际上非常简单, [参考文档](http://docs.sublimetext.info/en/latest/file_processing/build_systems.html)

sublime tex3中菜单`Tools` -> `Bulid System` -> `New Bulid System...`, 输入以下代码:

```json
{
    "cmd": ["D:\\Program Files\\tcc\\tcc.bat", "$file", "D:\\Program Files\\tcc\\tcc.exe"],
    "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
    "selector": "source.c"
}
```
    
这实际上是一段json数据, `cmd`后面的列表中是要执行的命令, 第一个参数数是一个`tcc.bat`脚本, 代码如下:
    
```bat
start cmd /k %2 -run %1
```
这段代码表示, 打开一个新的命令窗口, 执行`%2 -run %1`命令, 就是在你新的命令行窗口中运行`tcc -run $file`

将编译系统文件保存为`tcc.sublime-build`, 现在在菜单`Tools` -> `Bulid System`下面应该可以看到tcc菜单了

### 3. 测试运行
    
在sublime text3中随便编写一个C语言文件, 例:

```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main()
{
    printf("char的范围:%d~%d\n", CHAR_MIN, CHAR_MAX);
    printf("unsigned char的范围:0~%u\n", UCHAR_MAX);
    printf("short的范围:%d~%d\n", SHRT_MIN, SHRT_MAX);
    printf("unsigned short的范围:0~%u\n", USHRT_MAX);
    printf("int的范围:%d~%d\n", INT_MIN, INT_MAX);
    printf("unsigned int的范围:0~%u\n", UINT_MAX);
    printf("long的范围:%ld~%ld\n", LONG_MIN, LONG_MAX);
    printf("unsigned long的范围:0~%lu\n", ULONG_MAX);
    printf("long long的范围:%lld~%lld\n", LLONG_MIN, LLONG_MAX);
    printf("unsigned long long的范围:0~%llu\n\n", ULLONG_MAX);

    printf("float的最小非零值:%.3e\n", FLT_MIN);
    printf("flaot的最大值:%.3e\n", FLT_MAX);
    printf("double的最小非零值:%.3e\n", DBL_MIN);
    printf("double的最大值:%.3e\n", DBL_MAX);
    printf("long double的最小非零值:%.3le\n", LDBL_MIN);
    printf("long double的最大值:%.3le\n\n", LDBL_MAX);

    printf("flaot类型的十进制精度:%u\n", FLT_DIG);
    printf("double类型的十进制精度:%u\n", DBL_DIG);
    printf("long double的十进制精度:%u\n", LDBL_DIG);

    return 0;
}

```

快捷键`ctrl+b` 或者菜单`Tools` -> `Bulid System` -> `tcc`, 自动弹出命令行窗口, 并运行程序了:

![image](http://ozi7kdi5x.bkt.clouddn.com/2d22d417909f2c1.jpg)

### 4. 其他问题

乱码问题, 更改命令行代码页, [参考文章](http://blog.csdn.net/zhyh1435589631/article/details/51303756)


