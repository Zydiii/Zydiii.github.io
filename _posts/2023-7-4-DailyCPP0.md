---
layout: post
title: "DailyCPP（一）"
description: ""
date: 2023-7-4
feature_image: images/2023.7.4/0.png
tags: [C++, linux]
---

<!--more-->

## day0

### `ldd`

- `ldd` 用于打印程序或者库文件所依赖的共享库列表，`ldd` 不是一个可执行程序，而只是一个shell脚本，`ldd` 能够显示可执行模块的dependency(所属)(所属)，其原理是通过设置一系列的环境变量，如下：`LD_TRACE_LOADED_OBJECTS、LD_WARN、LD_BIND_NOW、LD_LIBRARY_VERSION、LD_VERBOSE` 等。当 `LD_TRACE_LOADED_OBJECTS` 环境变量不为空时，任何可执行程序在运行时，它都会只显示模块的dependency(所属)，而程序并不真正执行，其实质是通过 `ld-linux.so`（elf动态库的装载器）来实现的，`ld-linux.so` 模块会先于 executable 模块程序工作，并获得控制权，因此当上述的那些环境变量被设置时，`ld-linux.so` 选择了显示可执行模块的 dependency (所属)。 实际上可以直接执行 `ld-linux.so` 模块，如：`/lib/ld-linux.so.2 --list program`（这相当于`ldd program`）
- 在 `ldd` 的 manual 中提到，有些版本的 `ldd` 会执行 the program 去获取相关的依赖关系，所以永远不要随便对一个 untrusted executable 执行 `ldd`。作者提出为什么一个文本分析命令要去执行脚本呢，这个安全性问题很久之前就被提出了，但是并没有得到可靠的解决
- 那么为什么 `ldd` 会被设计得有如此风险呢，作者通过深入查看 `/usr/bin/ldd`，发现其就是一个 shell 脚本，它所做的事情就是将传入的 binary 传给一系列内置的 dynamic linkers，一旦找到相关的可以处理的 dynamic linker 之后，就会针对设置为 `LD_TRACE_LOADED_OBJECTS=1` 的 binary 运行 linker，linker 会将该 binary 沿着所需的直接和间接的库布置在新进程中，将信息转储到控制台然后退出，所以下面三种形式输出是一致的

```shell
$ ldd /bin/ls
$ /lib64/ld-linux-x86-64.so.2 --verify /bin/ls && echo 'Supported!'
Supported!
$ LD_TRACE_LOADED_OBJECTS=1 /lib64/ld-linux-x86-64.so.2 /bin/ls
linux-vdso.so.1 (0x00007fff5bbed000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f7786f48000)
	libcap.so.2 => /lib64/libcap.so.2 (0x00007f7786f3e000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f7786d60000)
	libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007f7786cc6000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f7786fad000)
```

- 另外有一些可替代的命令方案

    - `libtree the-binary`, which displays the direct and indirect libraries required by a binary as a tree.
    - `objdump -p the-binary | grep NEEDED`, which is the solution provided in the ldd(1) manual page I quoted earlier but is not equivalent to ldd because it can only handle direct dependencies.
    - `readelf -d the-binary | grep NEEDED`, which is a similar solution to the previous one with the same caveats.

### `ASLR`

- 是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，通过增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的




## 小结

## References

- [ldd(1) and untrusted binaries](https://jmmv.dev/2023/07/ldd-untrusted-binaries.html)