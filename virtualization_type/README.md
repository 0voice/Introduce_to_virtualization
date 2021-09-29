## 虚拟化架构
### 全虚拟化架构

![image](https://user-images.githubusercontent.com/87458342/134765255-5f9a2c30-d1a0-494a-b1c0-8bea17ed0eca.png)

客户机操作系统不宿主机操作系统的限制

### 操作系统层的虚拟化

![image](https://user-images.githubusercontent.com/87458342/134765287-ee5d3aed-d658-459e-a485-ea21b7595f07.png)

客户机操作系统必须要和宿主机操作系统保持一致

### 平台虚拟化（硬件虚拟化）

无需安装宿主机操作系统，客户机操作系统可以随意进行安装

![image](https://user-images.githubusercontent.com/87458342/134765323-7ea39d1f-4e93-4f27-8396-082e422edf10.png)

### Hypervisor
Hypervisor是一种运行在物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享一套基础物理硬件，因此也可以看作是虚拟环境中的“元”操作系统，他可以协调访问服务器上的所有的物理设备和虚拟机，也叫虚拟机监视器。Hypervisor是所有虚拟化技术的核心。当服务器启动并执行Hypervisor时，他会给每一台虚拟机分配适量的内存、CPU、网络和磁盘，并加载所有虚拟机的客户操作系统。常见的产品有Vmware、KVM、Xen等。

