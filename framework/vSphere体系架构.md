# vSphere 体系架构

## 1. vSphere 体系

![image](https://user-images.githubusercontent.com/87458342/134766160-6aae06d0-42a2-466c-a77b-88cf18c50bd2.png)

![image](https://user-images.githubusercontent.com/87458342/134766161-4d1f200a-0a95-408f-a962-6f6bf020906b.png)

### 1.1 VMware vSphere 6组件
#### 1.1.1 VMware ESXi
vSphere 早期版本存在ESX，ESX：虚拟机平台管理程序，ESX包含了一个VMware Kernel（虚拟化管理内核）和一个命令行式的Service Console（服务控制台）（vSphere 4.1将是最后一个包含ESX版本的平台，其后续版本仅将包含ESXi）
ESXi（又名vSphere Hypervisor）：基本功能同ESX，但ESXi仅保留管理内核（VMKernel）以及VMM而不再包含服务控制台（用vCLI 或 PowerCLI替代其大部分功能），所以体积很小，可安装在嵌入式设备如U盘上。

#### 1.1.2 VMware vCenter Server
虚拟化平台管理中心控制系统，有windows和linux版，

#### 1.1.3 vSphere Update Manager
vSphere环境升级，打补丁的工具，

#### 1.1.4 VMware vSphere Client和vSphere Web Client
vSphere客户端，

#### 1.1.5 vRealize Orchestrator（原名vCenter Orchestrator）
自动化引擎，建立工作流的工具---任务编排器

#### 1.1.6 VMware Data Protection
数据保护-----备份虚拟机

#### 1.1.7 vSphere with Operations Management（单独购买或购买套件）
监控和管理的工具

### 1.2 产品与特性
#### 1.2.1 VMware ESXi
#### 1.2.2 VMware vCenter Server
#### 1.2.3 vSphere Update Manager
#### 1.2.4 VMware vSphere Client and vSphere Web Client
#### 1.2.5 VMware vShield Zones
#### 1.2.6 VMware vCenter Orchestrator
#### 1.2.7 vSphere Virtual Symmetric Multi-Processing
#### 1.2.8 vSphere vMotion and Storage vMotion
#### 1.2.9 vSphere Distributed Resource Scheduler
#### 1.2.10 vSphere Storage DRS
#### 1.2.11 Storage I/O Control and Network I/O Control
#### 1.2.12 Profile-Driven Storage
#### 1.2.13 vSphere High Availability
#### 1.2.14 vSphere Fault Tolerance
#### 1.2.15 vSphere Storage APIs（应用程序接口）for Data Protection and VMware Data Recovery

### 1.3 必须使用vCenter Server才能支持的特性
#### 1.3.1 virtual machine templates（虚拟机模版）
#### 1.3.2 role-based access controls （基于角色的访问控制）
#### 1.3.3 fine-grained resource allocation controls （粒度的资源关联控制）
#### 1.3.4 VMware Vmotion （在线迁移）
#### 1.3.5 VMware Distributed Resource Scheduler （分布式资源调度）
#### 1.3.6 VMware High Availability （高可用性）
#### 1.3.7 VMware Fault Tolerance （容错）
#### 1.3.8 Enhanced VMotion Compatibility (EVC) （增强vMotion兼容性）
#### 1.3.9 Host profiles （主机脚本）
#### 1.3.10 vNetwork Distributed Switches （分布式交换机）
#### 1.3.11 Storage and Network I/O Control （存储和网络I/O控制）
#### 1.3.12 Sphere Storage DRS （存储分布式资源调度）

<br/>
VMware虚拟化拓扑图：
<br/>

![image](https://user-images.githubusercontent.com/87458342/134766310-b1e6e5ca-fd87-4a85-b53b-378eadc006e2.png)

## 2. 传统架构VS虚拟化
6.0与5.5版本的对比:
<br/>

![image](https://user-images.githubusercontent.com/87458342/134766331-9231357c-fb16-4136-926e-873b878ae450.png)

<br/>
传统物理基础架构：                                                   虚拟化架构：
<br/>

![image](https://user-images.githubusercontent.com/87458342/134766340-1950120e-22e2-4743-a898-484f38438ba6.png)

<br/>

![image](https://user-images.githubusercontent.com/87458342/134766347-f967a629-6fcf-4a7f-95ec-e4c9550c61e6.png)

<br/>

![image](https://user-images.githubusercontent.com/87458342/134766350-4c12f2f1-4efa-429e-adb8-1de7aba4f299.png)

<br/>

![image](https://user-images.githubusercontent.com/87458342/134766358-6d383c29-8659-4038-a3c8-41a65142cffa.png)

<br/>

![image](https://user-images.githubusercontent.com/87458342/134766359-b5412e79-5710-4ec0-8eb8-8e9ff0139587.png)

<br/>

![image](https://user-images.githubusercontent.com/87458342/134766366-aee40fef-152b-41a3-a669-f2eec08a2a98.png)

<br/>

![image](https://user-images.githubusercontent.com/87458342/134766371-031b4de8-0cd8-48dd-baa1-ef58cadae68c.png)

<br/>

![image](https://user-images.githubusercontent.com/87458342/134766374-3149314a-6d24-4cbd-9564-fbb9be8637dd.png)

<br/>

![image](https://user-images.githubusercontent.com/87458342/134766377-c2709065-e010-41eb-a400-c840b6490e18.png)

## 3. 虚拟化的优势
### 3.1 虚拟化优点：
优点1：提高硬件整合率：虚拟化使得低利用率的服务器负载整合到一台服务器，安全可靠地达到很高的硬件利用率
优点2：快速部署服务器
优点3：降低整体投资成本(TCO)将不同应用负载虚拟化使得用户可以大大减少服务器的数量典型的平均整合比率在8:1到15:1
优点4：节能降耗
优点5：提高了系统可用性物理主机被虚拟化后，计算资源均被池化。当资源池里一个节点发生故障时，运行在其上的虚拟机将自动迁移到健康的物理主机上。

## 4. VMware vSphere 方案概览
### 4.1 基于vSphere的虚拟数据中心基础架构
vSphere 可加快现有数据中心向云计算的转变，同时还支持兼容的公有云服务，从而为业界唯一的混合云模式奠定了基础。vSphere，许多群体称之为“ESXi”，即底层虚拟化管理程序体系结构的名称，这是一种采用尖端技术的裸机虚拟化管理程序。
vSphere 是市场上最先进的虚拟化管理程序，具有许多独特的功能和特性，其中包括：
#### 1.4.1.1 磁盘空间占用量小，因此可以缩小受攻击面并减少补丁程序数量
#### 1.4.1.2 不依赖操作系统，并采用加强型驱动程序
#### 1.4.1.3 具备高级内存管理功能，能够消除重复内存页或压缩内存页
#### 1.4.1.4 通过集成式的集群文件系统提供高级存储管理功能
#### 1.4.1.5 高I/O可扩展性可消除I/O瓶颈

![image](https://user-images.githubusercontent.com/87458342/134766410-c0a66c20-9d00-48a4-8dcc-a75360ae9105.png)

基于VMware vSphere 的虚拟数据中心由基本物理构建块（例如x86 虚拟化服务器、存储器网络和阵列、IP 网络、管理服务器和桌面客户端）组成。

### 4.2 vSphere 数据中心的物理拓扑
![image](https://user-images.githubusercontent.com/87458342/134766418-793cfa31-729a-4922-a97f-ef1529cc1a25.png)

vSphere 数据中心拓扑包括下列组件:
#### 1.4.2.6 计算服务器：ESXi主机群，在祼机上运行ESXi 的业界标准 x86 服务器。ESXi 软件为虚拟机提供资源，并运行虚拟机。每台计算服务器在虚拟环境中均称为独立主机。可以将许多配置相似的x86 服务器组合在一起，并与相同的网络和存储子系统连接，以便提供虚拟环境中的资源集合（称为群集）。
#### 1.4.2.7 存储网络和阵列光纤通道：SAN 阵列、iSCSI SAN 阵列和 NAS 阵列是广泛应用的存储技术，VMware vSphere支持这些技术以满足不同数据中心的存储需求。
#### 1.4.2.8 IP 网络：每台计算服务器都可以有多个物理网络适配器，为整个VMware vSphere 数据中心提供高带宽和可靠的网络连接。
#### 1.4.2.9 vCenter Server： vCenter Server 为数据中心提供一个单一控制点
#### 1.4.2.10 管理客户端：这些界面包括VMware vSphere Client (vSphere Client)、vSphere Web Client（用于通过 Web 浏览器访问）或 vSphere Command-Line Interface (vSphere CLI)。

### 4.3 ESXi架构和组件
如下图所示，从体系结构来说ESXi包含虚拟化层和虚拟机，而虚拟化层有两个重要组成部分：虚拟化管理程序VMkernel和虚拟机监视器VMM（守护进程）。ESXi主机可以通过vSphere Client、vCLI、API/SDK和CIM接口接入管理。
1. VMkernel 是虚拟化的核心和推动力，由VMware 开发并提供与其他操作系统提供的功能类似的某些功能，如进程创建和控制、信令、文件系统和进程线程。VMkernel控制和管理服务器的实际资源，它用资源管理器排定VM顺序，为它们动态分配CPU时间、内存和磁盘及网络访问。它还包含了物理服务器各种组件的设备驱动器——例如，网卡和磁盘控制卡、VMFS文件系统和虚拟交换机。VMkernel 专用于支持运行多个虚拟机及提供如下核心功能：
资源调度----->CPU、内存
I/O 堆栈----->网卡、存储
设备驱动程序------>网卡等

2. 每个 ESXi 主机的关键组件是一个称为VMM 的进程（守护进程）。对于每个已开启的虚拟机，将在VMkernel中运行一个 VMM。虚拟机开始运行时，控制权将转交给VMM，然后由 VMM 依次执行虚拟机发出的指令。VMkernel 将设置系统状态，以便VMM 可以直接在硬件上运行。然而，虚拟机中的操作系统并不了解此次控制权转交，而会认为自己是在硬件上运行。VMM 使虚拟机可以像物理机一样运行，而同时仍与主机和其他虚拟机保持隔离。因此，如果单台虚拟机崩溃，主机本身以及主机上的其他虚拟机将不受任何影响。

### 4.4 虚拟机的组件：操作系统、VMware Tools 以及虚拟资源和硬件。
#### 1.4.4.1 操作系统
虚拟机与所有标准x86 操作系统和应用程序完全兼容。在一台物理主机的不同虚拟机里，可以根据应用需求同时运行不同的x86操作系统，彼此之间不会冲突，且对x86操作系统无需进行任何修改。
#### 1.4.4.2 Vmware Tools
VMware Tools 是一套实用程序，能够提高虚拟机的客户操作系统的性能，并改善对虚拟机的管理。VMwareTools 服务是一项在客户操作系统内执行各种功能的服务。该服务在客户操作系统启动时自动启动。该服务可执行的功能包括：

①将消息从 ESXi 主机传送到客户操作系统。
②向ESXi主机发送心跳信号，使其知道客户操作系统正在运行。
③实现客户操作系统与主机操作系统之间的时间同步。
④在虚拟机中运行脚本并执行命令。
⑤为使用 VMware VIX API 创建的与客户操作系统绑定的调用提供支持，除Mac OS X 客户操作系统外。
⑥允许指针在 Windows 客户操作系统的客户机和Workstation 之间自由移动。
⑦帮助创建 Windows 客户操作系统中由特定备份应用程序使用的快照。
⑧在客户操作系统中安装 VMware Tools 后，它还会提供 VMware 设备驱动程序，包括 SVGA 显示驱动程序、用于某些客户操作系统的 vmxnet 网络连接驱动程序、用于某些客户操作系统的 BusLogic SCSI 或 LSI Logic驱动程序、用于在虚拟机之间进行有效内存分配的内存控制驱动程序、用于将 I/O 置于静默状态（使用VMware Data Recovery 或 VMware vStorage API for Data Recovery）以进行备份的同步驱动程序、用于实现文件夹共享的内核模块以及 VMware 鼠标驱动程序。
⑨各种驱动程序：

### 4.3 虚拟硬件
每个虚拟机都有虚拟硬件，这些虚拟硬件在所安装的客户操作系统及其应用中显示为物理硬件。每个客户操作系统都能识别出常规硬件设备，但它并不知道这些设备实际上是虚拟设备。虚拟机具有统一的硬件（少数选项可以由系统管理员控制）。统一硬件使得虚拟机可以跨vSphere 主机进行迁移













