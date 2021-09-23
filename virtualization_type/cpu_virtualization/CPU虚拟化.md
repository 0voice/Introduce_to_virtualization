# <h1> CPU虚拟化 </h1>

## <h2 id="cpu_virtualization_mode1">1. 基于二进制翻译的全虚拟化（Full Virtualization with Binary Translation） </h2>

![image](https://user-images.githubusercontent.com/87458342/134466367-8c6ec4cf-cb71-4c9d-95f2-a0f177eef1e4.png)

客户操作系统运行在 Ring 1，它在执行特权指令时，会触发异常（CPU的机制，没权限的指令会触发异常），然后 VMM 捕获这个异常，在异常里面做翻译，模拟，最后返回到客户操作系统内，客户操作系统认为自己的特权指令工作正常，继续运行。但是这个性能损耗，就非常的大，简单的一条指令，执行完，了事，现在却要通过复杂的异常处理过程。

异常 “捕获（trap）-翻译（handle）-模拟（emulate）” 过程：

![image](https://user-images.githubusercontent.com/87458342/134466413-87122124-2cc4-4195-80e0-05316d8494ea.png)

### <h2 id="cpu_virtualization_mode2">2. 超虚拟化（或者半虚拟化/操作系统辅助虚拟化 Paravirtualization）  </h2> 

半虚拟化的思想就是， 修改操作系统内核，替换掉不能虚拟化的指令， 通过超级调用（hypercall ）直接和底层的虚拟化层hypervisor 来通讯 ，hypervisor  同时也提供了超级调用接口来满足其他关键内核操作，比如内存管理、中断和时间保持。

这种做法省去了全虚拟化中的捕获和模拟，大大提高了效率。所以像XEN这种半虚拟化技术，客户机操作系统都是有一个专门的定制内核版本，和x86、mips、arm这些内核版本等价。这样以来，就不会有捕获异常、翻译、模拟的过程了，性能损耗非常低。这就是XEN这种半虚拟化架构的优势。这也是为什么XEN只支持虚拟化Linux，无法虚拟化windows原因，微软不改代码啊。

![image](https://user-images.githubusercontent.com/87458342/134466479-8f7b5aa3-6a36-42fa-b1d2-7ec66ca1f756.png)

### <h2 id="cpu_virtualization_mode3">3. 硬件辅助的虚拟化  </h2> 

2005年后，CPU厂商Intel 和 AMD 开始支持虚拟化了。 Intel 引入了 Intel-VT （Virtualization Technology）技术。 这种 CPU，有 VMX root operation 和 VMX non-root operation两种模式，两种模式都支持Ring 0 ~ Ring 3 共 4 个运行级别。这样，VMM 可以运行在 VMX root operation模式下，客户OS运行在VMX non-root operation模式下。

![image](https://user-images.githubusercontent.com/87458342/134466527-4eb1f3a0-bb69-48a2-bde0-d8a958e5f95e.png)

而且两种操作模式可以互相转换。运行在 VMX root operation 模式下的 VMM 通过显式调用 VMLAUNCH 或 VMRESUME 指令切换到 VMX non-root operation 模式，硬件自动加载 Guest OS 的上下文，于是 Guest OS 获得运行，这种转换称为 VM entry。Guest OS 运行过程中遇到需要 VMM 处理的事件，例如外部中断或缺页异常，或者主动调用 VMCALL 指令调用 VMM 的服务的时候（与系统调用类似），硬件自动挂起 Guest OS，切换到 VMX root operation 模式，恢复 VMM 的运行，这种转换称为 VM exit。VMX root operation 模式下软件的行为与在没有 VT-x 技术的处理器上的行为基本一致；而VMX non-root operation 模式则有很大不同，最主要的区别是此时运行某些指令或遇到某些事件时，发生 VM exit。

也就说，硬件这层就做了些区分，这样全虚拟化下，那些靠“捕获异常-翻译-模拟”的实现就不需要了。而且CPU厂商，支持虚拟化的力度越来越大，靠硬件辅助的全虚拟化技术的性能逐渐逼近半虚拟化，再加上全虚拟化不需要修改客户操作系统这一优势，全虚拟化技术应该是未来的发展趋势。
