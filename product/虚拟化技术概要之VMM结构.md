# 虚拟化技术概要之VMM结构

## 1. 概述

当前主流的 VMM (Virtual Machine Monitor)
实现结构可以分为三类：

宿主模型 (OS-hosted VMMs)
Hypervisor 模型 (Hypervisor
VMMs)
混合模型 (Hybrid VMMs)

## 2. 宿主模型

![image](https://user-images.githubusercontent.com/87458342/134775005-0a99d644-690d-44fc-9484-00da7807fe6c.png)

该结构的 VMM，物理资源由 Host OS (Windows, Linux etc.) 管理
实际的虚拟化功能由
VMM 提供，其通常是 Host OS 的独立内核模块（有的实现还含用户进程，如负责 I/O 虚拟化的用户态设备模型）
VMM 通过调用 Host OS
的服务来获得资源，实现 CPU，内存和 I/O 设备的虚拟化
VMM 创建出 VM 后，通常将 VM 作为 Host OS
的一个进程参与调度

如上图所示，VMM 模块负责 CPU 和内存虚拟化，由 ULM 请求 Host OS 设备驱动，实现 I/O
设备的虚拟化。

**优点**：可以充分利用现有 OS 的设备驱动，VMM 无需自己实现大量的设备驱动，轻松实现 I/O
设备的虚拟化。
**缺点**：因资源受 Host OS 控制，VMM 需调用 Host OS
的服务来获取资源进行虚拟化，其效率和功能会受到一定影响。

采用该结构的 VMM 有：VMware Workstation, VMWare
Server (GSX), Virtual PC, Virtual Server, KVM（早期）


3. Hypervisor 模型
![image](https://user-images.githubusercontent.com/87458342/134775036-024d2eb6-1717-4ac4-ab22-be5707fa8bec.png)

该结构中，VMM 可以看作一个为虚拟化而生的完整 OS，掌控有所有资源（CPU，内存，I/O 设备）
VMM
承担管理资源的重任，其还需向上提供 VM 用于运行 Guest OS，因此 VMM
还负责虚拟环境的创建和管理。

优点：因 VMM
同时具有物理资源的管理功能和虚拟化功能，故虚拟化的效率会较高；安全性方面，VM 的安全只依赖于 VMM 的安全
缺点：因
VMM 完全拥有物理资源，因此，VMM 需要进行物理资源的管理，包括设备的驱动，而设备驱动的开发工作量是很大的，这对 VMM
是个很大的挑战。

采用该结构的 VMM 有：VMWare ESX Server,  WindRiver Hypervisor, 
KVM（后期）

4. 混合模型
该结构是上述两种模式的混合体，VMM 依然位于最底层，拥有所有物理资源，但 VMM 会主动让出大部分 I/O 设备的控制权，将它们交由一个运行在特权 VM 上的特权 OS 来控制。
VMM
只负责 CPU 和内存的虚拟化，I/O 设备的虚拟化由 VMM 和特权 OS 共同完成：
![image](https://user-images.githubusercontent.com/87458342/134775048-11945c9d-39d5-4f8b-812b-3023b69a31f1.png)

优点：可利用现有 OS 的 I/O 设备驱动；VMM 直接控制 CPU
和内存等物理资源，虚拟化效率较高；若对特权 OS 的权限控制得当，虚拟机的安全性只依赖于 VMM。
缺点：因特权 OS
运行于 VM 上，当需要特权 OS 提供服务时，VMM 需要切换到特权 OS，这里面就产生上下文切换的开销。

采用该结构的 VMM 有：Xen,
SUN Logical Domain
























