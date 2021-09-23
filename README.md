<h1> 💨 500篇关于虚拟化的经典资料，含CPU虚拟化，磁盘虚拟化，内存虚拟化，io虚拟化。 </h1>

<br/>

<!--
![image](https://user-images.githubusercontent.com/87458342/134299929-8f7e5675-600e-4af3-b5b9-57b0bcf3f0af.png)
-->

<br/>

* [🪐 虚拟化基础](#nav_vt0) 
  * [🍃 虚拟化分类](#nav_vt0_chapter1)
  * [🦕 CPU虚拟化](#nav_vt1) 
  * [🦖 内存虚拟化](#nav_vt2)
  * [🐊 IO虚拟化](#nav_vt3)
  * [🦎 磁盘虚拟化](#nav_vt4)
* [🌱 宣讲&PPT](#nav_vt1_chapter2)
* [🧿 视频](#nav_vt1_chapter3)
* [🍀 论文](#nav_vt1_chapter4)
* [🌰 开源项目](#nav_vt1_chapter5)
* [📄 文章](#nav_vt1_chapter6)
* [📙 电子书籍](#nav_vt1_chapter7)

<br/>
<br/>
<br/>

# <h1 id="nav_vt0">🪐 虚拟化基础</h1>

## <h2 id="nav_vt0_chapter1">🍃 虚拟化分类</h2>

* [虚拟化技术分类](https://github.com/0voice/Introduce_to_virtualization/blob/main/virtualization_type/虚拟化技术分类.md)
* [虚拟化的分类](https://github.com/0voice/Introduce_to_virtualization/blob/main/virtualization_type/虚拟化技术分类.md)
* [全虚拟化和半虚拟化](https://github.com/0voice/Introduce_to_virtualization/blob/main/virtualization_type/全虚拟化和半虚拟化.md)
* [虚拟化五种类型](https://github.com/0voice/Introduce_to_virtualization/blob/main/virtualization_type/虚拟化五种类型.md)

## <h2 id="nav_vt1">🦕 CPU虚拟化 </h2>

### 1. 基于二进制翻译的全虚拟化（Full Virtualization with Binary Translation）

![image](https://user-images.githubusercontent.com/87458342/134466367-8c6ec4cf-cb71-4c9d-95f2-a0f177eef1e4.png)

客户操作系统运行在 Ring 1，它在执行特权指令时，会触发异常（CPU的机制，没权限的指令会触发异常），然后 VMM 捕获这个异常，在异常里面做翻译，模拟，最后返回到客户操作系统内，客户操作系统认为自己的特权指令工作正常，继续运行。但是这个性能损耗，就非常的大，简单的一条指令，执行完，了事，现在却要通过复杂的异常处理过程。

异常 “捕获（trap）-翻译（handle）-模拟（emulate）” 过程：

![image](https://user-images.githubusercontent.com/87458342/134466413-87122124-2cc4-4195-80e0-05316d8494ea.png)

### 2. 超虚拟化（或者半虚拟化/操作系统辅助虚拟化 Paravirtualization）

半虚拟化的思想就是， 修改操作系统内核，替换掉不能虚拟化的指令， 通过超级调用（hypercall ）直接和底层的虚拟化层hypervisor 来通讯 ，hypervisor  同时也提供了超级调用接口来满足其他关键内核操作，比如内存管理、中断和时间保持。

这种做法省去了全虚拟化中的捕获和模拟，大大提高了效率。所以像XEN这种半虚拟化技术，客户机操作系统都是有一个专门的定制内核版本，和x86、mips、arm这些内核版本等价。这样以来，就不会有捕获异常、翻译、模拟的过程了，性能损耗非常低。这就是XEN这种半虚拟化架构的优势。这也是为什么XEN只支持虚拟化Linux，无法虚拟化windows原因，微软不改代码啊。

![image](https://user-images.githubusercontent.com/87458342/134466479-8f7b5aa3-6a36-42fa-b1d2-7ec66ca1f756.png)

### 3. 硬件辅助的虚拟化

2005年后，CPU厂商Intel 和 AMD 开始支持虚拟化了。 Intel 引入了 Intel-VT （Virtualization Technology）技术。 这种 CPU，有 VMX root operation 和 VMX non-root operation两种模式，两种模式都支持Ring 0 ~ Ring 3 共 4 个运行级别。这样，VMM 可以运行在 VMX root operation模式下，客户OS运行在VMX non-root operation模式下。

![image](https://user-images.githubusercontent.com/87458342/134466527-4eb1f3a0-bb69-48a2-bde0-d8a958e5f95e.png)

而且两种操作模式可以互相转换。运行在 VMX root operation 模式下的 VMM 通过显式调用 VMLAUNCH 或 VMRESUME 指令切换到 VMX non-root operation 模式，硬件自动加载 Guest OS 的上下文，于是 Guest OS 获得运行，这种转换称为 VM entry。Guest OS 运行过程中遇到需要 VMM 处理的事件，例如外部中断或缺页异常，或者主动调用 VMCALL 指令调用 VMM 的服务的时候（与系统调用类似），硬件自动挂起 Guest OS，切换到 VMX root operation 模式，恢复 VMM 的运行，这种转换称为 VM exit。VMX root operation 模式下软件的行为与在没有 VT-x 技术的处理器上的行为基本一致；而VMX non-root operation 模式则有很大不同，最主要的区别是此时运行某些指令或遇到某些事件时，发生 VM exit。

## <h2 id="nav_vt2">🦖 内存虚拟化 </h2>
## <h2 id="nav_vt3">🐊 IO虚拟化 </h2>
## <h2 id="nav_vt4">🦎 磁盘虚拟化 </h2>
<br/>
<br/>

# <h1 id="nav_vt1_chapter2">🌱 宣讲&PPT</h1>

# <h1 id="nav_vt1_chapter3">🧿 视频</h1>

# <h1 id="nav_vt1_chapter4">🍀 论文</h1>

No.|Title|Translate|Company
:-------: | :--------------- | :------------|:---------------
1|[《Emerging Virtualization Technology》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/01_Emerging_Virtualization_Technology.pdf) |《新兴虚拟化技术》|
2|[《HYPERVISOR FOR VIRTUALIZATION IN PRIVATE CLOUD》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/02_HYPERVISOR_FOR_VIRTUALIZATION_IN_PRIVATE_CLOUD.pdf) |《私有云虚拟化管理程序》|
3|[《Secure Virtualization for Cloud Environment Using Hypervisor-based Technology》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/03_Secure_Virtualization_for_Cloud_Environment_Using_Hypervisor-based_Technology.pdf) |《基于虚拟机监控程序的云环境安全虚拟化技术》|
4|[《OPERATING SYSTEM VIRTUALIZATION IN THE EDUCATION OF COMPUTER SCIENCE STUDENTS》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/04_OPERATING_SYSTEM_VIRTUALIZATION_IN_THE_EDUCATION_OF_COMPUTER_SCIENCE_STUDENTS.pdf) |《计算机科学学生教育中的操作系统虚拟化》|
5|[《Virtualization Technologies and Cloud Security:advantages, issues, and perspectives》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/05_Virtualization_Technologies_and_Cloud_Security:advantages,issues,and_perspectives.pdf) |《虚拟化技术和云安全：优势、问题和前景》|
6|[《Xen and the Art of Virtualization》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/06_Xen_and_the_Art_of_Virtualization.pdf) |《Xen与虚拟化的艺术》|
7|[《Analysis of Virtualization Technologies for High Performance Computing Environments》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/07_Analysis_of_Virtualization_Technologies_for_High_Performance_Computing_Environments.pdf) |《高性能计算环境的虚拟化技术分析》|
8|[《Research on Cloud Computing Based on Storage Virtualization in Data Center》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/08_Research_on_Cloud_Computing_Based_on_Storage_Virtualization_in_Data_Center.pdf) |《基于数据中心存储虚拟化的云计算研究》|
9|[《Architecture for Technology Transformation》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/09_Architecture_for_Technology_Transformation.pdf) |《技术改造架构》|
10|[《A Study On Virtualization Techniques And Challenges In Cloud Computing》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/10_A_Study_On_Virtualization_Techniques_And_Challenges_In_Cloud_Computing.pdf) |《云计算中的虚拟化技术与挑战研究》|
11|[《Virtual Machine Security Guidelines Version 1.0》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/11_Virtual_Machine_Security_Guidelines_Version_1.0.pdf) |《虚拟机安全指南1.0版》|
12|[《Comparative Performance Analysis of the Virtualization Technologies in Cloud Computing》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/12_Comparative_Performance_Analysis_of_the_Virtualization_Technologies_in_Cloud_Computing.pdf) |《云计算中虚拟化技术的比较性能分析》|
13|[《Improving Business Performance by Employing VirtualizationTechnology: A Case Study in the Financial Sector》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/13_Improving_Business_Performance_by_Employing_VirtualizationTechnology:_A_Case_Study_in_the_Financial_Sector.pdf) |《利用虚拟化技术提高业务绩效：金融行业案例研究》|
14|[《Consolidation Using Oracle's SPARCVirtualization Technologies》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/14_Consolidation_Using_Oracle's_SPARCVirtualization_Technologies.pdf) |《使用Oracle的SPARCVirtualization技术进行整合》|
15|[《Development of a virtualization systems architecture course for Development of a virtualization systems architecture course for the information sciences and technologies depar the information sciences and technologies department at the tment at the Rochester Institute of Technology (RIT) Rochester Institute of Technology (RIT)》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/15-Development_of_a_virtualization_systems_architecture_course_for_t.pdf) |《为信息科学和技术开发虚拟化系统体系结构课程的虚拟化系统体系结构课程的开发》|
16|[《Educational Infrastructure Using Virtualization Technologies: Experience at Kaunas University of Technology》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/16_Educational_Infrastructure_Using_Virtualization_Technologies:_Experience_at_Kaunas_University_of_Technology.pdf) |《“利用虚拟化技术的教育基础设施：考纳斯技术大学的经验”》|
17|[《Comparative Study of Virtual Machine Software Packages with Real Operating System》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/17_Comparative_Study_of_Virtual_Machine_Software_Packages_with_Real_Operating_System.pdf) |《虚拟机软件包与真实操作系统的比较研究》|
18|[《Dell EMC Unity: Virtualization Integration》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/18_Dell_EMC_Unity:_Virtualization_Integration.pdf) |《Dell EMC Unity：虚拟化集成》|
19|[《A Study On Virtualization And Virtual Machines》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/19_A_Study_On_Virtualization_And_Virtual_Machines.pdf) |《虚拟化与虚拟机研究》|
20|[《Review on Virtualization for Cloud Computing》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/20_Review_on_Virtualization_for_Cloud_Computing.pdf) |《云计算虚拟化综述》|
21|[《A Survey on Virtualization and Hypervisor-based Technology in Cloud Computing Environment》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/21_A_Survey_on_Virtualization_and_Hypervisor-based_Technology_in_Cloud_Computing_Environment.pdf) |《云计算环境中基于虚拟化和虚拟机监控程序的技术综述》|
22|[《STUDY ON VIRTUALIZATION TECHNOLOGY AND ITS IMPORTANCE IN CLOUD COMPUTING ENVIRONMENT》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/22_STUDY_ON_VIRTUALIZATION_TECHNOLOGY_AND_ITS_IMPORTANCE_IN_CLOUD_COMPUTING_ENVIRONMENT.pdf) |《虚拟化技术及其在云计算环境中的重要性研究》|
23|[《Research on the Virtualization Technology in Cloud Computing Environment》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/23_Research_on_the_Virtualization_Technology_in_Cloud_Computing_Environment.pdf) |《云计算环境下虚拟化技术研究》|
24|[《Research and Development on Network Virtualization Technologies in Japan》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/24_Research_and_Development_on_Network_Virtualization_Technologies_in_Japan.pdf) |《日本网络虚拟化技术的研究与开发》|
25|[《Eliminate Software Development and Testing Constraints with Service Virtualization》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/25_Eliminate_Software_Development_and_Testing_Constraints_with_Service_Virtualization.pdf) |《通过服务虚拟化消除软件开发和测试限制》|
26|[《Network Virtualization: A Data Plane Perspective》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/26_Network_Virtualization:_A_Data_Plane_Perspective.pdf) |《网络虚拟化：数据平面透视图》|
27|[《A taxonomy of virtualization technologies》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/27_A_taxonomy_of_virtualization_technologies.pdf) |《虚拟化技术分类》|
28|[《Network Functions Virtualisation》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/28_Network_Functions_Virtualisation.pdf) |《网络功能虚拟化》|
29|[《Recommendations of the National Institute of Standards and Technology》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/29_Recommendations_of_the_National_Institute_of_Standards_and_Technology.pdf) |《国家标准与技术研究所建议》|
30|[《Big Data Virtualization: Why and How?》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/30_Big_Data_Virtualization:_Why_and_How.pdf) |《大数据虚拟化：为什么和如何？》|
31|[《Server Virtualization Technology and ltsLatest Trends》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/31_Server_Virtualization_Technology_and_ltsLatest_Trends.pdf) |《服务器虚拟化技术和最新趋势》|
32|[《Virtualization Technologies for Cars Solutions to increase safety and security of vehicular ECUs》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/32_Virtualization_Technologies_for_Cars_Solutions_to_increase_safety_and_security_of_vehicular_ECUs.pdf) |《提高车辆ECU安全性的车辆虚拟化技术解决方案》|
33|[《Virtualization and Future Technologies》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/33_Virtualization_and_Future_Technologies.pdf) |《虚拟化与未来技术》|
34|[《Virtualization and the Computer Architecture》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/34_Virtualization_and_the_Computer_Architecture.pdf) |《虚拟化与计算机体系结构》|
35|[《Virtualization Introduction QSM White Paper》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/35_Virtualization_Introduction_QSM_White_Paper.pdf) |《虚拟化简介QSM白皮书》|
36|[《Security Implications of Different Virtualization Approaches for Secure Cyber Architectures》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/36_Security_Implications_of_Different_Virtualization_Approaches_for_Secure_Cyber_Architectures.pdf) |《不同虚拟化方法对安全网络体系结构的安全影响》|
37|[《Server Virtualization: A Step Toward Cost Efficiency and Business Agility》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/37_Server_Virtualization_A_Step_Toward_Cost_Efficiency_and_Business_Agility.pdf) |《服务器虚拟化：迈向成本效益和业务灵活性的一步》|
38|[《Performance Implications of Virtualization》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/38_Performance_Implications_of_Virtualization.pdf) |《虚拟化的性能影响》|
39|[《State-of-the-Art of Virtualization, its Security Threats and Deployment Models》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/39_State-of-the-Art_of_Virtualization,_its_Security_Threats_and_Deployment_Models.pdf) |《虚拟化技术现状、安全威胁和部署模型》|
40|[《HMI & Virtualization in Process Automation》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/40_HMI_&_Virtualization_in_Process_Automation.pdf) |《过程自动化中的人机界面和虚拟化》|
41|[《Terra: A Virtual Machine-Based Platform for Trusted Computing》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/41_Terra_A_Virtual_Machine-Based_Platform_for_Trusted_Computing.pdf) |《Terra：基于虚拟机的可信计算平台》|
42|[《Research on Virtualization Technology for Real-time Reconfigurable Systems》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/42_Research_on_Virtualization_Technology_for_Real-time_Reconfigurable_Systems.pdf) |《实时可重构系统虚拟化技术研究》|
43|[《A Survey on Virtualization Technologies》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/43_A_Survey_on_Virtualization_Technologies.pdf) |《虚拟化技术概览》|
44|[《Intel Virtualization Technology》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/44_Intel_Virtualization_Technology.pdf) |《英特尔虚拟化技术》|
45|[《EXPERIENCES WITH VIRTUALIZATION TECHNOLOGY IN EDUCATION》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/45_EXPERIENCES_WITH_VIRTUALIZATION_TECHNOLOGY_IN_EDUCATION.pdf) |《虚拟化技术在教育中的应用经验》|
46|[《VIRTUALIZATION IN CLOUD COMPUTING》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/46_VIRTUALIZATION_IN_CLOUD_COMPUTING.pdf) |《云计算中的虚拟化》|
47|[《Systematic Study of Virtualization》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/47_Systematic_Study_of_Virtualization.pdf) |《虚拟化系统研究》|
48|[《Virtualization in Cloud Computing : Developments and Trends》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/48_Virtualization_in_Cloud_Computing__Developments_and_Trends.pdf) |《云计算中的虚拟化：发展与趋势》|
49|[《Virtualization Overview》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/49_Virtualization_Overview.pdf) |《虚拟化概述》|
50|[《ArcGIS Pro Virtualization》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/50_ArcGIS_Pro_Virtualization.pdf) |《ArcGIS Pro虚拟化》|
51|[《Intel® Virtualization Technology(VT) in Converged Application Platforms》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/51_Intel®_Virtualization_Technology(VT)_in_Converged_Application_Platforms.pdf) |《聚合应用程序平台中的英特尔虚拟化技术（VT）》|
52|[《Virtualization Technology Whitepaper - Infrastructure to Perform Static Tools and Binary Analysis》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/52_Virtualization_Technology_Whitepaper-Infrastructure_to_Perform_Static_Tools_and_Binary_Analysis.pdf) |《虚拟化技术白皮书-执行静态工具和二进制分析的基础架构》|
53|[《A Survey on Virtual Machine Security》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/53_A_Survey_on_Virtual_Machine_Security.pdf) |《虚拟机安全调查》|
54|[《Intel® Virtualization Technology: Hardware Support for Efficient Processor Virtualization》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/54_Intel®_Virtualization_Technology_Hardware_Support_for_Efficient_Processor_Virtualization.pdf) |《英特尔®虚拟化技术：高效处理器虚拟化的硬件支持》|
55|[《Network functions virtualization》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/55_Network_functions_virtualization.pdf) |《网络功能虚拟化》|
56|[《BEYOND VIRTUALIZATION The MontaVista Approach to Multi-core SoC Resource Allocation and Control》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/56_BEYOND_VIRTUALIZATION_The_MontaVista_Approach_to_Multi-core_SoC_Resource_Allocation_and_Control.pdf) |《超越虚拟化——多核SoC资源分配和控制的MontaVista方法》|
57|[《A PRINCIPLED TECHNOLOGIES WHITE PAPER》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/57_A_PRINCIPLED_TECHNOLOGIES_WHITE_PAPER.pdf) |《原则性技术白皮书》|
58|[《Data Virtualization – Flexible Technology for the Agile Enterprise》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/58_Data_Virtualization–Flexible_Technology_for_the_Agile_Enterprise.pdf) |《数据虚拟化——敏捷企业的灵活技术》|
59|[《Top 5 Things You Need in a Virtualization Management Solution》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/59_Top_5_Things_You_Need_in_a_Virtualization_Management_Solution.pdf) |《虚拟化管理解决方案中需要的五大要素》|
60|[《The IBM Advantage for Implementing the Virtualization Reference Architecture》](https://github.com/0voice/Introduce_to_virtualization/blob/main/papers/60_The_IBM_Advantage_for_Implementing_the_Virtualization_Reference_Architecture.pdf) |《IBM实施虚拟化参考体系结构的优势》|


# <h1 id="nav_vt1_chapter5">🌰 开源项目</h1>

## <h3 id="nav_vt1_chapter5_01">[KVM](http://www.linux-kvm.org)</h3>

KVM (全称是 Kernel-based Virtual Machine) 是 Linux 下 x86 硬件平台上的全功能虚拟化解决方案，包含一个可加载的内核模块 kvm.ko 提供和虚拟化核心架构和处理器规范模块。
使用 KVM 可允许多个包括 Linux 和 Windows 每个虚拟机有私有的硬件，包括网卡、磁盘以及图形适配卡等。

## <h3 id="nav_vt1_chapter5_02">[Xen](https://xenproject.org)</h3>

Xen 是一个开放源代码虚拟机监视器，由剑桥大学开发。它打算在单个计算机上运行多达100个满特征的操作系统。操作系统必须进行显式地修改（“移植”）以在Xen上运行（但是提供对用户应用的兼容性）。这使得Xen无需特殊硬件支持，就能达到高性能的虚拟化。

## <h3 id="nav_vt1_chapter5_03">[OpenVZ](https://openvz.livejournal.com)</h3>

OpenVZ是基于Linux内核和作业系统的操作系统级虚拟化技术。OpenVZ允许物理服务器运行多个操作系统，被称虚拟专用服务器（VPS，Virtual Private Server）或虚拟环境（VE, Virtual Environment）。
与VMware这种虚拟机和Xen这种半虚拟化技 术相比，OpenVZ的host OS和guest OS都必需是Linux（虽然在不同的虚拟环境里可以用不同的Linux发行版）。但是，OpenVZ声称这样做有性能上的优势。根据OpenVZ网站的 说法，使用OpenVZ与使用独立的服务器相比，性能只会有1-3%的损失。
OpenVZ是SWsoft, Inc.公司开发的专有软件Virtuozzo的基础。OpenVZ的授权为GPLv2。
OpenVZ由两部分组成，一个经修改过的操作系统核心与及用户工具。

## <h3 id="nav_vt1_chapter5_04">[VirtualBox](https://www.virtualbox.org)</h3>

VirtualBox 是一款功能强大的 x86 虚拟机软件，它不仅具有丰富的特色，而且性能也很优异。更可喜的是，VirtualBox 于数日前走向开源，成为了一个发布在 GPL 许可之下的自由软件。

## <h3 id="nav_vt1_chapter5_05">[Lguest](http://lguest.ozlabs.org)</h3>

Lguest 是由IBM工程师Rusty Russell（澳大利亚开发者)发起的虚拟化项目，是一个只有5000行代码的精简hypervisor（虚拟机管理程序），它已经包括在最近版本的内核里了。和KVM相似，它支持 Intel和AMD芯片的最新虚拟化技术。但又与VMware公司的ESX Server不同，在Lguest创建的虚拟机里的操作系统知道自己是被虚拟出来的。所以在调用CPU周期时它可以直接向真正的硬件发出请求，而不是作为中间媒介而降低了效率，因此这种架构大大提高了效率。Lguest采用GPL授权。

* [VManagePlatform ：一个KVM虚拟化管理平台](https://github.com/Moniter123/VManagePlatform)
* [MalAnalyzer ：基于docker虚拟化的恶意代码沙箱](https://github.com/felicitychou/MalAnalyzer)
* [PinVMP ：虚拟化代码辅助分析工具](https://github.com/lmy375/pinvmp)
* [File-Management ：基于虚拟磁盘模仿ext2的图形化文件管理系统](https://github.com/pancerZH/File-Management)


# <h2 id="nav_vt1_chapter6">📄 文章</h2>

<div align=left>

知识体系|宣讲&PPT|视频|论文|开源项目|文章|电子书籍
:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:
[🌴](#nav_vt1_chapter1)|[🌱](#nav_vt1_chapter2)|[🧿](#nav_vt1_chapter3)|[🍀](#nav_vt1_chapter4) |[🌰](#nav_vt1_chapter5)|[📄](#nav_vt1_chapter6)|[📙](#nav_vt1_chapter7)

</div>

# <h1 id="nav_vt1_chapter7">📙 电子书籍</h1>

* 《VMware vSphere4 云操作系统搭建配置入门与实战》.pdf
* 《VMwareCertifiedProfessionalTest Prep》.pdf
* 《企业虚拟化实战Vmware篇》.pdf
* 《精通VMware vSphere 5原版》.pdf
* 《虚拟智慧VMware vSphere运维实录》.pdf


<br/>
<br/>
