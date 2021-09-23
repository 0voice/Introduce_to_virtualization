# KVM 内存虚拟化

## 1 内存虚拟化的概念

除了 CPU 虚拟化，另一个关键是内存虚拟化，通过内存虚拟化共享物理系统内存，动态分配给虚拟机。虚拟机的内存虚拟化很象现在的操作系统支持的虚拟内存方式，应用程序看到邻近的内存地址空间，这个地址空间无需和下面的物理机器内存直接对应，操作系统保持着虚拟页到物理页的映射。现在所有的 x86 CPU 都包括了一个称为内存管理的模块MMU（Memory Management Unit）和 TLB(Translation Lookaside Buffer)，通过MMU和TLB来优化虚拟内存的性能。
 
KVM 实现客户机内存的方式是，利用mmap系统调用，在QEMU主线程的虚拟地址空间中申明一段连续的大小的空间用于客户机物理内存映射。

![image](https://user-images.githubusercontent.com/87458342/134482796-f8c0f8e9-621e-4411-8e61-ac65c47e4a82.png)
（图片来源 HVA 同下面的 MA，GPA 同下面的 PA，GVA 同下面的 VA）

在有两个虚机的情况下，情形是这样的：

![image](https://user-images.githubusercontent.com/87458342/134482834-d7317132-d737-4448-a124-ff203096dd73.png)

可见，KVM 为了在一台机器上运行多个虚拟机，需要增加一个新的内存虚拟化层，也就是说，必须虚拟 MMU 来支持客户操作系统，来实现 VA -> PA -> MA 的翻译。客户操作系统继续控制虚拟地址到客户内存物理地址的映射 （VA -> PA），但是客户操作系统不能直接访问实际机器内存，因此VMM 需要负责映射客户物理内存到实际机器内存 （PA -> MA）。
 
VMM 内存虚拟化的实现方式：
* 软件方式：通过软件实现内存地址的翻译，比如 Shadow page table （影子页表）技术
* 硬件实现：基于 CPU 的辅助虚拟化功能，比如 AMD 的 NPT 和 Intel 的 EPT 技术 

* 影子页表技术：
![image](https://user-images.githubusercontent.com/87458342/134482914-1c8ebc1f-2236-442b-8e16-601f5b0843ab.png)


## 2 KVM 内存虚拟化

KVM 中，虚机的物理内存即为 qemu-kvm 进程所占用的内存空间。KVM 使用 CPU 辅助的内存虚拟化方式。在 Intel 和 AMD 平台，其内存虚拟化的实现方式分别为：
* AMD 平台上的 NPT （Nested Page Tables） 技术
* Intel 平台上的 EPT （Extended Page Tables）技术
EPT 和 NPT采用类似的原理，都是作为 CPU 中新的一层，用来将客户机的物理地址翻译为主机的物理地址。关于 EPT， Intel 官方文档中的技术如下
![image](https://user-images.githubusercontent.com/87458342/134482998-082002da-1385-4953-8ec9-0b4c449a4d86.png)

EPT的好处是，它的两阶段记忆体转换，特点就是将 Guest Physical Address → System Physical Address，VMM不用再保留一份 SPT (Shadow Page Table)，以及以往还得经过 SPT 这个转换过程。除了降低各部虚拟机器在切换时所造成的效能损耗外，硬体指令集也比虚拟化软体处理来得可靠与稳定。

## 3 KSM （Kernel SamePage Merging 或者 Kernel Shared Memory）

KSM 在 Linux 2.6.32 版本中被加入到内核中。

## 3.1 原理

其原理是，KSM 作为内核中的守护进程（称为 ksmd）存在，它定期执行页面扫描，识别副本页面并合并副本，释放这些页面以供它用。因此，在多个进程中，Linux将内核相似的内存页合并成一个内存页。这个特性，被KVM用来减少多个相似的虚拟机的内存占用，提高内存的使用效率。由于内存是共享的，所以多个虚拟机使用的内存减少了。这个特性，对于虚拟机使用相同镜像和操作系统时，效果更加明显。但是，事情总是有代价的，使用这个特性，都要增加内核开销，用时间换空间。所以为了提高效率，可以将这个特性关闭。

## 3.2 好处
其好处是，在运行类似的客户机操作系统时，通过 KSM，可以节约大量的内存，从而可以实现更多的内存超分，运行更多的虚机。 

## 3.3 合并过程

（1）初始状态：
![image](https://user-images.githubusercontent.com/87458342/134483197-ec89d15e-e44a-4b22-80ce-9688702668ce.png)

（2）合并后：
![image](https://user-images.githubusercontent.com/87458342/134483230-1cead6d2-af31-4710-9280-52ac9af0972c.png)

（3）Guest 1 写内存后：
![image](https://user-images.githubusercontent.com/87458342/134483260-b69643da-e7e1-4cf8-97f1-5509f39b4b87.png)

## 4  KVM Huge Page Backed Memory （巨页内存技术）

这是KVM虚拟机的又一个优化技术.。Intel 的 x86 CPU 通常使用4Kb内存页，当是经过配置，也能够使用巨页(huge page): (4MB on x86_32, 2MB on x86_64 and x86_32 PAE)

使用巨页，KVM的虚拟机的页表将使用更少的内存，并且将提高CPU的效率。最高情况下，可以提高20%的效率！

使用方法，需要三部：

```C
mkdir /dev/hugepages
mount -t hugetlbfs hugetlbfs /dev/hugepages
```
* 保留一些内存给巨页
```C
sysctl vm.nr_hugepages=2048 （使用 x86_64 系统时，这相当于从物理内存中保留了2048 x 2M = 4GB 的空间来给虚拟机使用）
```
* 给 kvm 传递参数 hugepages
```C
qemu-kvm - qemu-kvm -mem-path /dev/hugepages
```
也可以在配置文件里加入：
<memoryBacking>
<hugepages/>
</memoryBacking>
验证方式，当虚拟机正常启动以后，在物理机里查看：
```C
cat /proc/meminfo |grep -i hugepages
```
老外的一篇文档，他使用的是libvirt方式，先让libvirtd进程使用hugepages空间，然后再分配给虚拟机。




原文作者： SammyLiu





