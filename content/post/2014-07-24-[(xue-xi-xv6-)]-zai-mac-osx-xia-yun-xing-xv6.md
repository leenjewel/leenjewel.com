---
title: "【学习 xv6 】在 Mac OSX 下运行 xv6"
date: 2014-07-24 22:14:15 +0800
draft: true
categories: [xv6]
---

## 编译 xv6

首先要用 git 把 xv6 的源码 clone 到本地

```text
git clone git://pdos.csail.mit.edu/xv6/xv6.git
```

xv6 源代码中的 README 文件中 Building And Running xv6 章节有这么一段说明：“To build xv6 on an x86 ELF machine（like Linux or FreeBSD）, run "make"......”

这里打算简单解释一下什么是 ELF machine 。按理说我们的计算机运行的程序（这里主要指二进制程序）例如 Windows 下的 exe 文件这样的其实无非都是计算机指令。如果试想一下，我们在同一台计算机上，相同架构下的 CPU 对应可识别的应该也是同一套指令，也就是说不管是 Windows 下的还是 Linux 亦或是 Mac OS 下的二进制程序所对应的 CPU 指令都应该是一样的。那为什么这几个操作系统间的二进制可执行程序不能通用呢？

答案简单的解释起来就是：“他们之间各自的二进制可执行程序的组织方式不同”，也即所谓的格式不统一。

一个二进制程序除了包含有给 CPU 去执行的指令外，还有一些其他的信息，比如数据段，版本信息，链接指示信息等，它们与代码指令一起组成了二进制的可执行程序。而到底以什么样的格式或者顺序将他们组织在一起，每个操作系统就各有不同了，这也是造成他们之间的二进制程序无法通用的主要原因。

言归正传，这里的 ELF 就指的是产生的二进制程序的组装方式。 ELF 是 Linux 和 FreeBSD 这种类 Unix 系统的二进制程序组装格式。 Windows 下的有 PE、COFF 格式，当然 Mac OSX 也有自己的格式。

说到这里对 ELF、PE、COFF 等二进制程序是如何编译链接组装的，强烈向大家推荐一本书[《程序员的自我修养--链接、装载与库》](http://product.china-pub.com/195439)书中对这方面知识的介绍相当的详细和深入！

继续回到我们的 xv6 。前面说了这么多其实是想告诉大家在 Mac OSX 上编译 xv6 绝非容易的事。当然在 xv6 项目的 README 文件中也给了相关的解法。简单的说就是让你按照[这个页面的步骤](http://pdos.csail.mit.edu/6.828/2012/tools.html)去安装相应的编译工具，可以理解为在 Mac OSX 上通过交叉编译来支持产生 ELF 格式的二进制程序（就好比通过 Xcode 可以交叉编译出在 ARM 架构的 iPhone 上运行的 APP 一样）。

但是我在尝试着按照[这个页面的步骤](http://pdos.csail.mit.edu/6.828/2012/tools.html)去安装相应的编译工具时失败了。于是我放弃了这条道路转而“曲线救国”了。我们手头装个 [VirtualBox](http://www.virtualbox.org) 虚拟机然后再在虚拟机上安装一个 Linux 发型版这件事相对容易多了。没错，就是这样，我是在我的 Linux 虚拟机上编译好 xv6 然后再存到 Mac OSX 的文件夹下的。

编译过程非常简单，直接在 xv6 项目文件夹下执行

```shell
make
```

编译成功后我们会得到 xv6.img 和 fs.img 两个文件。接下来说说如何运行我们编译好的 xv6。

## 运行 xv6

按照官方的说法，xv6 支持在 qemu 虚拟机下运行。在 Linux 环境安装好 qemu 虚拟机后可以在命令行执行

```shell
qemu-system-i386 -serial mon:stdio -hdb fs.img xv6.img -smp 1 -m 512
```

或者执行

```shell
qemu-system-x86_64 -smp 1 -parallel stdio -hdb fs.img xv6.img -m 512
```

都可以成功的把 xv6 运行起来。但是在 Mac OSX 下这却是一件“头疼”的事儿。在 Mac OSX 下通过源码编译的方式安装 qemu 虚拟机本身就非常容易失败。xv6 本身又无法在 [VirtualBox](http://www.virtualbox.org) 下运行。不过几经查找我发现了这么一款虚拟机，名字叫 [Q](http://www.kju-app.org/) 是个基于 qemu 的 Mac OSX 下的虚拟机软件，免费而且开源，酷！

把 [Q](http://www.kju-app.org/) 这款虚拟机下载下来安装好，打开之后我们就可以配置我们的 xv6 虚拟机了。

在 [Q](http://www.kju-app.org/) 里点击 + 号新建一个虚拟机，在弹出的虚拟机配置对话框里 General 里给这个虚拟机起个名字，比如就叫“xv6”

在 Hardware 配置页里设置 Platform 为 x86 PC 1CPU

在 Hardware 配置页里设置 RAM 为 512 MB，这里默认的配置是 128 MB，但是 128 MB 启动 xv6 会报错。至于为什么 128 MB 内存无法正常运行 xv6 等我学习研究了 xv6 的代码后在告诉大家。

在 Hardware 配置页的 Hard disk 里把 xv6.img 载入进去。

在 Advanced 配置页的 Hard disk 2 里把 fs.img 载入进去。

大功告成，赶快运行你的虚拟机一睹 xv6 的芳容吧！
