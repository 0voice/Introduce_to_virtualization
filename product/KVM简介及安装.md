# KVM简介及安装

## 1. KVM 介绍

### 1.1 虚拟化简史
![image](https://user-images.githubusercontent.com/87458342/134773817-acc4b715-2427-47c4-9862-898ca2ed910c.png)

其中，KVM 全称是 基于内核的虚拟机（Kernel-based Virtual Machine），它是Linux 的一个内核模块，该内核模块使得 Linux 变成了一个 Hypervisor：

它由 Quramnet 开发，该公司于 2008年被 Red Hat 收购。
它支持 x86 (32 and 64 位), s390, Powerpc 等 CPU。
它从 Linux 2.6.20 起就作为一模块被包含在 Linux 内核中。
它需要支持虚拟化扩展的 CPU。
它是完全开源的。官网。
本文介绍的是基于 X86 CPU 的 KVM。

### 1.2 KVM 架构

 KVM 是基于虚拟化扩展（Intel VT 或者 AMD-V）的 X86 硬件的开源的 Linux 原生的全虚拟化解决方案。KVM 中，虚拟机被实现为常规的 Linux 进程，由标准 Linux 调度程序进行调度；虚机的每个虚拟 CPU 被实现为一个常规的 Linux 线程。这使得 KMV 能够使用 Linux 内核的已有功能。
但是，KVM 本身不执行任何硬件模拟，需要用户空间程序通过 /dev/kvm 接口设置一个客户机虚拟服务器的地址空间，向它提供模拟 I/O，并将它的视频显示映射回宿主的显示屏。目前这个应用程序是 QEMU。

Linux 上的用户空间、内核空间和虚机：
![image](https://user-images.githubusercontent.com/87458342/134773827-1483ef53-592b-4b49-a4cf-73824fad5509.png)

* Guest：客户机系统，包括CPU（vCPU）、内存、驱动（Console、网卡、I/O 设备驱动等），被 KVM 置于一种受限制的 CPU 模式下运行。
* KVM：运行在内核空间，提供 CPU 和内存的虚级化，以及客户机的 I/O 拦截。Guest 的 I/O 被 KVM 拦截后，交给 QEMU 处理。
* QEMU：修改过的被 KVM 虚机使用的 QEMU 代码，运行在用户空间，提供硬件 I/O 虚拟化，通过 IOCTL /dev/kvm 设备和 KVM 交互。

#### KVM 是实现拦截虚机的 I/O 请求的原理：

现代 CPU 本身实现了对特殊指令的截获和重定向的硬件支持，甚至新硬件会提供额外的资源来帮助软件实现对关键硬件资源的虚拟化从而提高性能。以 X86 平台为例，支持虚拟化技术的 CPU  带有特别优化过的指令集来控制虚拟化过程。通过这些指令集，VMM 很容易将客户机置于一种受限制的模式下运行，一旦客户机试图访问物理资源，硬件会暂停客户机运行，将控制权交回给 VMM 处理。VMM 还可以利用硬件的虚级化增强机制，将客户机在受限模式下对一些特定资源的访问，完全由硬件重定向到 VMM 指定的虚拟资源，整个过程不需要暂停客户机的运行和 VMM 的参与。由于虚拟化硬件提供全新的架构，支持操作系统直接在上面运行，无需进行二进制转换，减少了相关的性能开销，极大简化了VMM的设计，使得VMM性能更加强大。从 2005 年开始，Intel 在其处理器产品线中推广 Intel Virtualization Technology 即 IntelVT 技术。

#### QEMU-KVM：

其实 QEMU 原本不是 KVM 的一部分，它自己就是一个纯软件实现的虚拟化系统，所以其性能低下。但是，QEMU 代码中包含整套的虚拟机实现，包括处理器虚拟化，内存虚拟化，以及 KVM需要使用到的虚拟设备模拟（网卡、显卡、存储控制器和硬盘等）。 为了简化代码，KVM 在 QEMU 的基础上做了修改。VM 运行期间，QEMU 会通过 KVM 模块提供的系统调用进入内核，由 KVM 负责将虚拟机置于处理的特殊模式运行。当虚机进行 I/O 操作时，KVM 会从上次系统调用出口处返回 QEMU，由 QEMU 来负责解析和模拟这些设备。   从 QEMU 角度看，也可以说是 QEMU 使用了 KVM 模块的虚拟化功能，为自己的虚机提供了硬件虚拟化加速。除此以外，虚机的配置和创建、虚机运行所依赖的虚拟设备、虚机运行时的用户环境和交互，以及一些虚机的特定技术比如动态迁移，都是 QEMU 自己实现的。  

#### KVM：
KVM 内核模块在运行时按需加载进入内核空间运行。KVM 本身不执行任何设备模拟，需要 QEMU 通过 /dev/kvm 接口设置一个 GUEST OS 的地址空间，向它提供模拟的 I/O 设备，并将它的视频显示映射回宿主机的显示屏。它是KVM 虚机的核心部分，其主要功能是初始化 CPU 硬件，打开虚拟化模式，然后将虚拟客户机运行在虚拟机模式下，并对虚机的运行提供一定的支持。以在 Intel 上运行为例，KVM 模块被加载的时候，它：
1. 首先初始化内部的数据结构；
2. 做好准备后，KVM 模块检测当前的 CPU，然后打开 CPU 控制及存取 CR4 的虚拟化模式开关，并通过执行 VMXON 指令将宿主操作系统置于虚拟化模式的根模式；
3. 最后，KVM 模块创建特殊设备文件 /dev/kvm 并等待来自用户空间的指令。

接下来的虚机的创建和运行将是 QEMU 和 KVM 相互配合的过程。两者的通信接口主要是一系列针对特殊设备文件 /dev/kvm 的 IOCTL 调用。其中最重要的是创建虚机。它可以理解成KVM 为了某个特定的虚机创建对应的内核数据结构，同时，KVM 返回一个文件句柄来代表所创建的虚机。
 
针对该句柄的调用可以对虚机做相应地管理，比如创建用户空间虚拟地址和客户机物理地址、真实物理地址之间的映射关系，再比如创建多个 vCPU。KVM 为每一个 vCPU 生成对应的文件句柄，对其相应地 IOCTL 调用，就可以对vCPU进行管理。其中最重要的就是“执行虚拟处理器”。通过它，虚机在 KVM 的支持下，被置于虚拟化模式的非根模式下，开始执行二进制指令。在非根模式下，所有敏感的二进制指令都被CPU捕捉到，CPU 在保存现场之后自动切换到根模式，由 KVM 决定如何处理。
 
除了 CPU 的虚拟化，内存虚拟化也由 KVM 实现。实际上，内存虚拟化往往是一个虚机实现中最复杂的部分。CPU 中的内存管理单元 MMU 是通过页表的形式将程序运行的虚拟地址转换成实际物理地址。在虚拟机模式下，MMU 的页表则必须在一次查询的时候完成两次地址转换。因为除了将客户机程序的虚拟地址转换了客户机的物理地址外，还要将客户机物理地址转化成真实物理地址。 

## 2. KVM 的功能列表

KVM 所支持的功能包括：
* 支持 CPU 和 memory 超分（Overcommit）
* 支持半虚拟化 I/O （virtio）
* 支持热插拔 （cpu，块设备、网络设备等）
* 支持对称多处理（Symmetric Multi-Processing，缩写为 SMP ）
* 支持实时迁移（Live Migration）
* 支持 PCI 设备直接分配和 单根 I/O 虚拟化 （SR-IOV）
* 支持 内核同页合并 （KSM ）
* 支持 NUMA （Non-Uniform Memory Access，非一致存储访问结构 ）

## 3. KVM 工具集合

* libvirt：操作和管理KVM虚机的虚拟化 API，使用 C 语言编写，可以由 Python,Ruby, Perl, PHP, Java 等语言调用。可以操作包括 KVM，vmware，XEN，Hyper-v, LXC 等在内的多种 Hypervisor。
* Virsh：基于 libvirt 的 命令行工具 （CLI）
* Virt-Manager：基于 libvirt 的 GUI 工具
* virt-v2v：虚机格式迁移工具
* virt-* 工具：包括 Virt-install （创建KVM虚机的命令行工具）， Virt-viewer （连接到虚机屏幕的工具），Virt-clone（虚机克隆工具），virt-top 等
* sVirt：安全工具

## 4. RedHat Linux KVM 安装

RedHat 有两款产品提供 KVM 虚拟化：
* Red Hat Enterprise Linux：适用于小的环境，提供数目较少的KVM虚机。最新的版本包括 6.5 和 7.0.
* Red Hat Enterprise Virtualization (RHEV)：提供企业规模的KVM虚拟化环境，包括更简单的管理、HA，性能优化和其它高级功能。最新的版本是 3.0.
 
RedHat Linux KVM:
* KVM 由 libvirt API 和基于该 API的一组工具进行管理和控制
* KVM 支持系统资源超分，包括内存和CPU的超分。RedHat Linux 最多支持物理 CPU 内核总数的10倍数目的虚拟CPU，但是不支持在一个虚机上分配超过物理CPU内核总数的虚拟CPU。
支持 KSM （Kenerl Same-page Merging 内核同页合并）

RedHat Linux KVM 有如下两种安装方式：

### 4.1 在安装  RedHat Linux 时安装 KVM

选择安装类型为 Virtualizaiton Host ：
![image](https://user-images.githubusercontent.com/87458342/134773830-cf93cc1c-b183-4f0a-b994-6a369d9a9e26.png)

可以选择具体的 KVM 客户端、平台和工具：
![image](https://user-images.githubusercontent.com/87458342/134773834-b8bcb144-ba01-4ed2-b5d0-f4217dd52796.png)

### 4.2 在已有的 RedHat Linux 中安装 KVM

这种安装方式要求该系统已经被注册，否则会报错：

```C
[root@rh65 ~]# yum install qemu-kvm qemu-img
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Setting up Install Process
Nothing to do
```

你至少需要安装 qemu-kvm qemu-img 这两个包。

```C
# yum install qemu-kvm qemu-img
```

你还可以安装其它工具包：
```C
# yum install virt-manager libvirt libvirt-python python-virtinst libvirt-client
```

### 4.3 QEMU/KVM 代码下载编译安装
#### 4.3.1 QEMU/KVM 的代码结构
QEMU/KVM 的代码包括几个部分：
* （1）KVM 内核模块是 Linux 内核的一部分。通常 Linux 比较新的发行版（2.6.20+）都包含了 KVM 内核，也可以从这里得到。比如在我的RedHat 6.5 上：
```C
[root@rh65 isoimages]# uname -r
2.6.32-431.el6.x86_64
[root@rh65 isoimages]# modprobe -l | grep kvm
kernel/arch/x86/kvm/kvm.ko
kernel/arch/x86/kvm/kvm-intel.ko
kernel/arch/x86/kvm/kvm-amd.ko
```
* （2）用户空间的工具即 qemu-kvm。qemu-kvm 是 KVM 项目从 QEMU 新拉出的一个分支（看这篇文章）。在 QEMU 1.3 版本之前，QEMU 和 QEMU-KVM 是有区别的，但是从 2012 年底 GA 的 QEMU 1.3 版本开始，两者就完全一样了。
* （3）Linux Guest OS virtio 驱动，也是较新的Linux 内核的一部分了。
* （4）Windows Guest OS virtio 驱动，可以从这里下载。

#### 4.3.2 安装 QEMU

RedHat 6.5 上自带的 QEMU 太老，0.12.0 版本，最新版本都到了 2.* 了。
* （1）. 参考 这篇文章，将 RedHat 6.5 的 ISO 文件当作本地源

```C
mount -o loop soft/rhel-server-6.4-x86_64-dvd.iso /mnt/rhel6/

vim /etc/fstab
=> /root/isoimages/soft/RHEL6.5-20131111.0-Server-x86_64-DVD1.iso /mnt/rhel6 iso9660 ro,loop
[root@rh65 qemu-2.3.0]# cat /etc/yum.repos.d/local.repo
[local]
name=local
baseurl=file:///mnt/rhel6/
enabled=1
gpgcjeck=0


  yum clean all<br>yum update

```

* （2）. 安装依赖包包
```C
yum install gcc
yum install autoconf
yum install autoconf automake libtool
yum install -y glib*
yum install zlib*
```

* （3）. 从 http://wiki.qemu.org/Download 下载代码，上传到我的编译环境 RedHat 6.5.
```C
tar -jzvf qemu-2.3.0.tar.bz2
cd qemu-2.3.0
./configure
make -j 4
make install
```

* （4）. 安装完成
```C
[root@rh65 qemu-2.3.0]# /usr/local/bin/qemu-x86_64 -version
qemu-x86_64 version 2.3.0, Copyright (c) 2003-2008 Fabrice Bellard
```

* （5）. 为方便起见，创建一个link
```C
ln -s /usr/bin/qemu-system-x86_64 /usr/bin/qemu-kvm
```

#### 4.3.3 安装 libvirt
可以从 libvirt 官网下载安装包。最新的版本是 0.10.2. 

## 5. 创建 KVM 虚机的几种方式
### 5.1 使用 virt-install 命令
```C
virt-install \
--name=guest1-rhel5-64 \
--file=/var/lib/libvirt/images/guest1-rhel5-64.dsk \
--file-size=8 \
--nonsparse --graphics spice \
--vcpus=2 --ram=2048 \
--location=http://example1.com/installation_tree/RHEL5.6-Serverx86_64/os \
--network bridge=br0 \
--os-type=linux \
--os-variant=rhel5.4
```

### 5.2 使用 virt-manager 工具
![image](https://user-images.githubusercontent.com/87458342/134773840-c620e427-152b-4f74-a4d3-3577adda5c90.png)

使用 VMM GUI 创建的虚机的xml 定义文件在 /etc/libvirt/qemu/ 目录中。

### 5.3 使用 qemu-img 和 qemu-kvm 命令行方式安装
（1）创建一个空的qcow2格式的镜像文件
```C
qemu-img create -f qcow2 windows-master.qcow2 10G
```
（2）启动一个虚机，将系统安装盘挂到 cdrom，安装操作系统
```C
qemu-kvm  -hda  windows-master.qcow2  -m  512  -boot d  -cdrom /home/user/isos/en_winxp_pro_with_sp2.iso
```

（3）现在你就拥有了一个带操作系统的镜像文件。你可以以它为模板创建新的镜像文件。使用模板的好处是，它会被设置为只读所以可以免于破坏。
```C
qemu-img create -b windows-master.qcow2 -f  qcow2   windows-clone.qcow2
```

（4）你可以在新的镜像文件上启动虚机了
```C
qemu-kvm  -hda  windows-clone.qcow2  -m 400
```

### 5.4 通过 OpenStack Nova 使用 libvirt API 通过编程方式来创建虚机





























