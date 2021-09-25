# Neutron实现网络虚拟化

## 1. 云计算时代数据中心物理网络的问题
数据中心虚拟化成为了趋势，最典型的场景莫过于：对数据中心的服务器进行虚拟化，来提高资源利用率，同时降低单位能耗。
但是，随着数据中心虚拟化程度的不断提高、虚拟化服务器规模的不断扩大，带来了巨大的管理压力。===>这正是云计算诞生的原因。
在大规模虚拟化的基础上，实现了自动化管理和集中化管理，就是云计算的基本模型。

云计算的超大规模加上其特有的服务模式带来了诸多亟需解决的问题，这些问题中，首当其冲的就是网络问题

我们需要从两个角度去看：
* 数据中心现有的物理网络，无法承载云计算机的超大规模
（详见1.1小节）
* 数据中心现有的物理网络，无法满足云计算SDN的要求
（详见1.2小节）

### 1.1 数据中心现有的物理网络，无法承载云计算机的超大规模
互联网行业数据中心的基本特征就是服务器的规模偏大。进入云计算时代后，其业务特征变得更加复杂，包括：虚拟化支持、多业务承载、资源灵活调度等（如图1所示）。与此同时，互联网云计算的规模不但没有缩减，反而更加庞大。这就给云计算的网络带来了巨大的压力。

![image](https://user-images.githubusercontent.com/87458342/134756305-8354dd6b-32c0-4dde-954e-50e0206a9054.png) <br/>
图1. 互联网云计算的业务特点

#### 大容量的MAC表项和ARP表项
虚拟化会导致更大的MAC表项。假设一个互联网云计算中心的服务器有5000台，按照1:20的比例进行虚拟化，则有10万个虚拟机。通常每个虚拟机会配置两个业务网口，这样这个云计算中心就有20万个虚拟网口，对应的就是需要20万个MAC地址和IP地址。云计算要求资源灵活调度，业务资源任意迁移。也就是说任意一个虚拟机可以在整个云计算网络中任意迁移。这就要求全网在一个统一的二层网络中。全网任意交换机都有可能学习到全网所有的MAC表项。与此对应的则是，目前业界主流的接入交换机的MAC表项只有32K，基本无法满足互联网云计算的需求。另外，网关需要记录全网所有主机、所有网口的ARP信息。这就需要网关设备的有效ARP表项超过20万。大部分的网关设备芯片都不具备这种能力。

#### 4K VLAN Trunk问题
传统的大二层网络支持任意VLAN的虚拟机迁移到网络的任意位置，一般有两种方式。方式一：虚拟机迁移后，通过自动化网络管理平台动态的在虚拟机对应的所有端口上下发VLAN配置；同时，还需要动态删除迁移前虚拟机对应所有端口上的VLAN配置。这种方式的缺点是实现非常复杂，同时自动化管理平台对多厂商设备还面临兼容性的问题，所以很难实现。方式二：在云计算网络上静态配置VLAN，在所有端口上配置VLAN trunk all。这种方式的优点是非常简单，是目前主流的应用方式。但这也带来了巨大的问题：任一VLAN内如果出现广播风暴，则全网所有VLAN内的虚拟机都会受到风暴影响，出现业务中断。

#### 4K VLAN上限问题
云计算网络中有可能出现多租户需求。如果租户及业务的数量规模超出VLAN的上限（4K），则无法支撑客户的需求。

#### 虚拟机迁移网络依赖问题
VM迁移需要在同一个二层域内，基于IP子网的区域划分限制了二层网络连通性的规模。

### 1.2 数据中心现有的物理网络，无法满足云计算SDN的要求
站着使用角度去考虑，比如我们使用阿里云
* 一方面：每个申请阿里云的人或组织都是一个独立的租户，而每个申请者自以为自己独享的网络资源，实际上是与其他申请者共享阿里云的物理网络，这就需要阿里云做好租户与租户直接的隔离，这是一个很重要的方面，显然，数据中心现有的物理网络无法满足需求： 数据中心现有的物理网络是固定的，需要手动配置的，单一的。
* 另一方面：就是自服务，试想，我们作为阿里云的使用者，完全可以构建自己的网络环境，这就需要云环境对租户提供API，从而满足租户基于云计算虚拟出来的网络之上来设计、部署租户自己需要的网络架构。
所以总结下来，云计算SDN的需求无非两点：
1. 租户“共享同一套物理网络”并且一定要实现“互相隔离”
2. 自服务

### 1.3 小结
总结1.1和1.2：
1. 其实很明显的可以看出来，无论是1.1还是1.2的问题，都指向了同一点：数据中心现有的物理网络好使，不够6。如何解决呢？
2. 简答地说，就是在数据中心现有的物理网络基础之上叠加一层我们自己的虚拟网络

## 2. 如何解决问题：Neutron实现网络虚拟化

### 2.1 Neutron组件介绍

neutron包含组件：
* neutron-server
* neutron-plugin
* neutron-agent

![image](https://user-images.githubusercontent.com/87458342/134756371-1d97e17a-4407-4af4-8e07-7095a8322fff.png)

neutron各组件功能介绍：

1. Neutron-server可以理解为一个专门用来接收Neutron REST API调用的服务器，然后负责将不同的rest api分发到不同的neutron-plugin上。
2. Neutron-plugin可以理解为不同网络功能实现的入口，各个厂商可以开发自己的plugin。Neutron-plugin接收neutron-server分发过来的REST API，向neutron database完成一些信息的注册，然后将具体要执行的业务操作和参数通知给自身对应的neutron agent。
3. Neutron-agent可以直观地理解为neutron-plugin在设备上的代理，接收相应的neutron-plugin通知的业务操作和参数，并转换为具体的设备级操作，以指导设备的动作。当设备本地发生问题时，neutron-agent会将情况通知给neutron-plugin。
4. Neutron database，顾名思义就是Neutron的数据库，一些业务相关的参数都存在这里。
5. Network provider，即为实际执行功能的网络设备，一般为虚拟交换机（OVS或者Linux Bridge）。


neutron-plugin分为core-plugin和service-plugin两类。
Core-plugin，Neutron中即为ML2（Modular Layer 2），负责管理L2的网络连接。ML2中主要包括network、subnet、port三类核心资源，对三类资源进行操作的REST API被neutron-server看作Core API，由Neutron原生支持。其中：

![image](https://user-images.githubusercontent.com/87458342/134756390-0340750d-51bc-4725-bd94-b62e0b6d5cde.png)

Service-plugin，即为除core-plugin以外其它的plugin，包括l3 router、firewall、loadbalancer、VPN、metering等等，主要实现L3-L7的网络服务。这些plugin要操作的资源比较丰富，对这些资源进行操作的REST API被neutron-server看作Extension API，需要厂家自行进行扩展。
“Neutron对Quantum的插件机制进行了优化，将各个厂商L2插件中独立的数据库实现提取出来，作为公共的ML2插件存储租户的业务需求，使得厂商可以专注于L2设备驱动的实现，而ML2作为总控可以协调多厂商L2设备共同运行”。在Quantum中，厂家都是开发各自的Service-plugin，不能兼容而且开发重复度很高，于是在Neutron中就为设计了ML2机制，使得各厂家的L2插件完全变成了可插拔的，方便了L2中network资源扩展与使用。

（注意，以前厂商开发的L2 plugin跟ML2都存在于neutron/plugins目录下，而可插拔的ML2设备驱动则存在于neutron/plugins/ml2/drivers目录下）

ML2作为L2的总控，其实现包括Type和Mechanism两部分，每部分又分为Manager和Driver。Type指的是L2网络的类型（如Flat、VLAN、VxLAN等），与厂家实现无关。Mechanism则是各个厂家自己设备机制的实现，如下图所示。当然有ML2，对应的就可以有ML3，不过在Neutron中L3的实现只负责路由的功能，传统路由器中的其他功能（如Firewalls、LB、VPN）都被独立出来实现了，因此暂时还没有看到对ML3的实际需求。

![image](https://user-images.githubusercontent.com/87458342/134756404-548fe4ad-c8f3-4dc6-b54b-f2338e4876d9.png)

一般而言，neutron-server和各neutron-plugin部署在控制节点或者网络节点上，而neutron agent则部署在网络节点上和计算节点上。我们先来分析控制端neutron-server和neutron-plugin的工作，然后再分析设备端neutron-agent的工作。

neutron新进展(dragon  flow):  https://www.ustack.com/blog/neutron-dragonflow/

### 2.2 Neturon网络虚拟化简介

想要了解openstack的虚拟化网络，必先知道承载openstack虚拟化网络的物理网络    

在实际的数据中心中，与openstack相关的物理网络可以分为三层：

1. OpenStack Cloud network：OpenStack 架构的网络
2. 机房intranet （external network）：数据中心所管理的的公司网（Intranet） ，虚机使用的 Floating IP 是这个网络的地址的一部分。
3. 以及真正的外部网络即 Internet：由各大电信运营商所管理的公共网络，使用公共IP。

External 网络和Internet 之间是数据中心的 Uplink 路由器，它负责通过 NAT 来进行两个网络之间的IP地址（即 floating IP 和 Internet/Public IP）转换，因此，这部分的控制不在 OpenStack 控制之下，我们无需关心它。

openstack架构的网络--->机房的网络external-----(NAT)--->外网internet

openstack架构的网络又可细分为以下几种（下述指的仍然是物理网络）：

1. 管理网络：包含api网络（public给外部用，admin给管理员用-是内部ip，internal给内部用-是内部ip）
2. 数据网络
3. 存储网络
4. IDRAC网络
5. PXE网络

数据网络主要用于vm之间的通信，这部分是neutron所实现网络虚拟化的核心，neutron基于该物理网络之上，来实现满足多租户要求的虚拟网络和服务。

Neutron 提供的网络虚拟化能力包括：

1. 二层到七层网络的虚拟化：L2（virtual switch）、L3（virtual Router 和 LB）、L4-7（virtual Firewall ）等
2. 网络连通性：二层网络和三层网络
3. 租户隔离性
4. 网络安全性
5. 网络扩展性
6. REST API
7. 更高级的服务，包括 LBaaS，FWaaS，VPNaaS 等。具体以后再慢慢分析。

Neutron主要为租户提供虚拟的
1. 交换机（switch）
2. 网络(L2 network)和子网（subnet），
3. 路由器（router）
4. 端口（port）

### 2.3 构建大二层网络：虚拟交换机（switch）+物理交换机

1. 什么是二层网络？    
二层指的是数据链路层，计算机与计算机之间的通信采用的是基于以太网协议广播的方式，请参考http://www.cnblogs.com/linhaifeng/articles/5937962.html

2. 什么是大二层网络？
大二层的概念指的是openstack中所有的vm都处于一个大的二层网络中，大二层也可以被想象成一堆二层交换机串联到一起。

3. 为什么要构建大二层网络？
构建大二层网络的目的就是为了满足vm可以迁移到全网的任意位置。二层无需网关，无需路由，因而资源调用更加灵活，反之，如果所有vm不在一个大二层中，那么vm迁移到另外一个位置（另外一个网络中），则需要我们人为地指定网关，添加路由策略，然而这还只是针对一台vm的迁移，要满足所有的vm的动态迁移，再去指定网关、路由等就不现实了。

4. 如何构建大二层网络？
一些列二层设备串联起来，组成一个大二层网络，在openstack中二层设备分为：物理的二层设备和虚拟机的二层设备
虚拟的二层设备又分为：
* 1. Neutron 默认采用的 开源Open vSwitch 创建的虚机交换机
* 2. Linux bridge。

![image](https://user-images.githubusercontent.com/87458342/134756485-7273167a-f719-4ee7-9d17-c894f76d9dbd.png)


需要特别强调的是：
1. 大二层网络是由虚拟二层设备和物理二层设备共同组成的；
2. 而这里面所有的二层设备合在一起相当于一个大的交换机；
3. 大二层网络是在计算节点或者网络节点启动时就会建立，而后期用户建立的网络，都是在此网络的基础之上进行逻辑划分，用户每新建一个网络都被分配相应的段ID（相当于vlan id）

### 2.4 大二层网络创建完毕后，用户如何使用？->网络(L2 network)、子网（subnet）
云平台在构建完毕后，即在物理节点都正常启动后，大二层网络也就创建好了，对用户来说，此时相当于一台大的二层交换机摆在面前，即用户创建网络就是把整个大二层网络当成一个交换机
然后用户需要做的是基于该网络划分自己的vlan，即建立一个l2 network，
例如：用户（例如demo用户）可以在自己的project下创建网络（L2 network），该网络就是在大二层的基础上划分出来的，是一个隔离的二层网段。类似于物理网络世界中的虚拟 LAN (VLAN)，每建立一个l2 network，都会被分配一个段ID，该段ID标识一个广播域，这个ID是被随机分配的，除非使用管理员身份在管理员菜单下，才可以手动指定该ID
那么就让我们在demo租户下以demo用户身份创建demo-net网络：

![image](https://user-images.githubusercontent.com/87458342/134756598-ea10273a-866d-41b1-b017-6a69001d46d3.png)

以admin用户登录，这样才能看到新建网络demo-net的段ID，其实此时demo-net仅仅只是一种逻辑上划分

![image](https://user-images.githubusercontent.com/87458342/134756602-cdc32604-954c-4fa4-b6b2-7142c3a49a87.png)

补充：使用管理员身份在管理员菜单下，创建网络可以手动指定段ID

![image](https://user-images.githubusercontent.com/87458342/134756615-cb44e92c-b739-4833-b514-e19a4ef386f8.png)

子网是一组 IPv4 或 IPv6 地址以及与其有关联的配置。它是一个地址池，OpenStack 可从中向虚拟机 (VM) 分配 IP 地址。每个子网指定为一个无类别域间路由 (Classless Inter-Domain Routing) 范围，必须与一个网络相关联。除了子网之外，租户还可以指定一个网关、一个域名系统 (DNS) 名称服务器列表，以及一组主机路由。这个子网上的 VM 实例随后会自动继承该配置。 

在 OpenStack Kilo 版本之前，用户需要自己输入 CIDR。Kilo 版本中实现了这个 Blueprint，使得 Neutron 能够从用户指定的 CIDR Pool 中自动分配 CIDR。

![image](https://user-images.githubusercontent.com/87458342/134756623-570f45b5-1cf7-43f0-b630-2cad6569c408.png)

![image](https://user-images.githubusercontent.com/87458342/134756624-3a957800-c216-413a-a27c-b4de58bb4068.png)

查看新建子网的信息  

![image](https://user-images.githubusercontent.com/87458342/134756626-f86491b8-7fb6-47d0-a174-eb0b1c54dfec.png)


创建的结果：就好比我们拿来一台“交换机”分出了一个VLAN.
注：在AWS中，该概念对应其 Subnet 概念。AWS 对 Subnet 的数目有一定的限制，默认地每个 VPC 内最多只能创建 200 个 Subnet。从CIDR角度看，一个 VPC 有一个比较大的网段，其中的每个 Subnet 分别拥有一个较小的网段。这种做法和OpenStack 有区别，OpenStack 中创建网络时不需要指定 CIDR。
在建立了子网后，会在网络节点上新增一个名称空间用来运行dhcp-agent专门为demo-net下的demo-subnet分配ip

* demo-net有一个段ID：51
* demo-subnet有网络ID：a603fd2c-35ef-46de-b9c4-da48680eb27d
* 连接dhcp-agent的端口网络ID：a603fd2c-35ef-46de-b9c4-da48680eb27d

![image](https://user-images.githubusercontent.com/87458342/134756648-ab717322-3a82-4196-8e25-5b08145b2ef0.png)

对应大二层网络上的变化为：在网络节点上新增dhcp-agent，在网络节点上的br-int新增端口，该端口的属于demo-net，及属于段ID：51
ps：新建子网后，便会网络节点新增dhcp-agent（解释一下：图中虚拟机是新建虚拟机后才有的）

![image](https://user-images.githubusercontent.com/87458342/134756658-bdef0289-fb82-4227-8981-6cabc9a491c2.png)

![image](https://user-images.githubusercontent.com/87458342/134756660-c58f121f-a7b0-41c7-a86e-fdbb68b4878f.png)

在demo-net新增一个子网的话，不会新增dhcp-agent，会新增地址池

![image](https://user-images.githubusercontent.com/87458342/134756665-e898840f-b027-41a3-b8fc-8f32619fed9a.png)

总结：

DHCP 服务是网络环境中必须有的。Neutron 提供基于 Dnamasq （轻型的dns和dhcp服务）实现的虚机 DHCP 服务，向租户网络内的虚机动态分配固定 IP 地址。它具有以下特性：

一个网络可以有多个运行在不同物理网络节点上的 DHCP Agent，同时向网络内的虚机提供服务
一个 DHCP Agent 只属于一个网络，在网络节点上运行在一个 network namespace 内
网络内的子网共享该 DHCP Agent

在demo-net以及demo-subnet建立成功后，就可以新建虚拟机连接到demo-net了，相当于同一个vlan中的俩台虚拟机，二层互通
此处我们新建vm1和vm2，网络拓扑如下   

![image](https://user-images.githubusercontent.com/87458342/134756695-8f565f16-e483-4a01-97a0-1e7e245fd7e0.png)

按照同样的套路创建demo-net1，它与demo-net是隔离的
![image](https://user-images.githubusercontent.com/87458342/134756712-84ebd444-381b-46f7-bbdf-d9448b11492c.png)

对应大二层网络的端口连接连接
![image](https://user-images.githubusercontent.com/87458342/134756730-d756fcf4-6056-42ef-af15-703892d12375.png)

这里所谓的隔离，可以理解为几个含义：
* 1. 每个网络使用自己的 DHCP Agent，每个 DHCP Agent 在一个 Network namespace 内
* 2. 不同网络内的IP地址可以重复（overlapping）
* 3. 跨网络的子网之间的流量必须走 L3 Virtual Router

### 2.5 虚拟机路由器（Vitual Router）
要想让demo-net1上的vm3与demo-net上的vm2通信，用户必须建立连接这两个网络的路由器router

![image](https://user-images.githubusercontent.com/87458342/134756784-abc2f0c9-cbbe-4206-b92d-ac632067ce35.png)

添加接口

![image](https://user-images.githubusercontent.com/87458342/134756805-7ff40614-a8d8-4e22-8285-b9fec8af8714.png)

查看网络拓扑

![image](https://user-images.githubusercontent.com/87458342/134756823-01f36950-874d-4f08-a0a7-67fe361eea1c.png)

对应大二层网络的变化

测试来自demo-net的vm2:172.16.45.6与demo-net1的vm3:172.16.18.3的连通性

![image](https://user-images.githubusercontent.com/87458342/134756834-dd12d390-c115-43e3-92b5-33afacb7bde1.png)

要想连接外网，需要用管理员身份，创建与外部网络绑定的网络，然后将demo租户下的test-router的网关指定为该网络

![image](https://user-images.githubusercontent.com/87458342/134756838-bf811524-334e-413d-ba38-b2df68ddc28b.png)

![image](https://user-images.githubusercontent.com/87458342/134756843-a94c77ac-a787-434a-9e4f-423ed8913d7b.png)

网络拓扑

![image](https://user-images.githubusercontent.com/87458342/134756851-83bca230-93ea-4ad1-88fc-2cfd2bea7910.png)

测试：

![image](https://user-images.githubusercontent.com/87458342/134756859-d19bd323-aa9e-4982-9d93-0f5e39dbaad8.png)

总结：一个 Virtual router 提供不同网段之间的 IP 包路由功能，由 Nuetron L3 agent 负责管理。

### 2.6 端口(Port)

一个 Port 代表虚拟网络交换机（logical network switch）上的一个虚机交换端口（virtual switch port）。虚机的网卡（VIF - Virtual Interface）会被连接到 port 上。当虚机的 VIF 连接到 Port 后，这个 vNIC 就会拥有 MAC 地址和 IP 地址。Port 的 IP 地址是从 subnet 中分配的。

## 3. 实现网络虚拟化的关键：如何构建虚拟的大二层网络

根据创建网络的用户的权限，Neutron L2 network 可以分为：

* Provider network：管理员创建的和物理网络有直接映射关系的虚拟网络。

![image](https://user-images.githubusercontent.com/87458342/134756891-e00fadf3-4a21-4d9d-8b51-43a0d8945c3f.png)

![image](https://user-images.githubusercontent.com/87458342/134756893-71cc3fea-f528-4531-8b1b-cd8cff3cfb46.png)

* Tenant network：租户普通用户创建的网络，物理网络对创建者透明，其配置由 Neutron根据管理员在系统中的配置决定。

![image](https://user-images.githubusercontent.com/87458342/134756900-c570cbaf-da05-46a7-b9c6-e20b4c85ee13.png)

虚机可以直接挂接到 provider network 或者 tenant network 上  “VMs can attach directly to both tenant and provider networks, and to networks with any provider:network_type value, assuming their tenant owns the network or its shared.”。

根据网络的类型，Neutron L2 network 可以分为：
* Flat network：基于不使用 VLAN 的物理网络实现的虚拟网络。每个物理网络最多只能实现一个虚拟网络。
* local network（本地网络）：一个只允许在本服务器内通信的虚拟网络，不知道跨服务器的通信。主要用于单节点上测试。
* VLAN network（虚拟局域网） ：基于物理 VLAN 网络实现的虚拟网络。共享同一个物理网络的多个 VLAN 网络是相互隔离的，甚至可以使用重叠的 IP 地址空间。每个支持 VLAN network 的物理网络可以被视为一个分离的 VLAN trunk，它使用一组独占的 VLAN ID。有效的 VLAN ID 范围是 1 到 4094。
* GRE network （通用路由封装网络）：一个使用 GRE 封装网络包的虚拟网络。GRE 封装的数据包基于 IP 路由表来进行路由，因此 GRE network 不和具体的物理网络绑定。
* VXLAN network（虚拟可扩展网络）：基于 VXLAN 实现的虚拟网络。同 GRE network 一样， VXLAN network 中 IP 包的路由也基于 IP 路由表，也不和具体的物理网络绑定。

注：在AWS中，该概念对应 VPC 概念。AWS 对 VPC 的数目有一定的限制，比如每个账户在每个 region 上默认最多只能创建 5 个VPC，通过特别的要求最多可以创建 100 个。

L2 network之Provider与Tenant的区别

* Provider network 是由 Admin 用户创建的，而 Tenant network 是由 tenant 普通用户创建的。
* Provider network 和物理网络的某段直接映射，比如对应某个 VLAN，因此需要预先在物理网络中做相应的配置。而 tenant network 是虚拟化的网络，Neutron 需要负责其路由等三层功能。
* 对 Flat 和 VLAN 类型的网络来说，只有 Provider network 才有意义。即使是这种类型的 tenant network，其本质上也是对应于一个实际的物理段。
* 对 GRE 和 VXLAN 类型的网络来说，只有 tenant network 才有意义，因为它本身不依赖于具体的物理网络，只是需要物理网络提供 IP 和 组播即可。
* Provider network 根据 admin 用户输入的物理网络参数创建；而 tenant work 由 tenant 普通用户创建，Neutron 根据其网络配置来选择具体的配置，包括网络类型，物理网络和 segmentation_id。
* 创建 Provider network 时允许使用不在配置项范围内的 segmentation_id。

### 3.2 依赖物理的二层来建立虚拟的大二层（vlan模式）
物理的二层指的是：物理网络是二层网络，基于以太网协议的广播方式进行通信
虚拟的二层指的是：neutron实现的虚拟的网络也是二层网络（openstack的vm机所用的网络必须是大二层），也是基于以太网协议的广播方式进行通信，但毫无疑问的是该虚拟网络依赖于物理的二层网络
物理二层+虚拟二层的典型代表：vlan网络模式，点击进入详解

### 3.3 依赖物理的三层来建立虚拟的大二层（gre模块与vxlan模式）
物理的三层指的是：物理网络是三层网络，基于ip路由的方式进行通信
虚拟的二层指的是：neutron实现的虚拟的网络仍然是二层网络（openstack的vm机所用的网络必须是大二层）,仍然是基于以太网协议的广播方式进行通信，但毫无疑问的是该虚拟网络依赖于物理的三层网络，这有点类似于VPN的概念，根本原理就是将私网的包封装起来，最终打上隧道的ip地址传输。
物理三层+虚拟二层的典型代表：gre模式与vxlan模式，点击进入详解














































