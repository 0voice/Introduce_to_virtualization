# OpenStack介绍及原理

## 什么是openstack
OpenStack 是一系列开源工具（或开源项目）的组合，主要使用池化虚拟资源来构建和管理私有云及公共云。其中的六个项目主要负责处理核心云计算服务，包括计算、网络、存储、身份和镜像服务。还有另外十多个可选项目，用户可把它们捆绑打包，用来创建独特、可部署的云架构。

## 云计算模式
1. IaaS：基础设施即服务（个人比较习惯的）：用户通过网络获取虚机、存储、网络，然后用户根据自己的需求操作获取的资源
2. PaaS：平台即服务：将软件研发平台作为一种服务， 如Eclipse/Java编程平台，服务商提供编程接口/运行平台等
3. SaaS：软件即服务 ：将软件作为一种服务通过网络提供给用户，如web的电子邮件、HR系统、订单管理系统、客户关系系统等。用户无需购买软件，而是向提供商租用基于web的软件，来管理企业经营活动

## OpenStack 中有哪些项目？
OpenStack 架构由大量开源项目组成。其中包含 6 个稳定可靠的核心服务，用于处理计算、网络、存储、身份和镜像； 同时，还为用户提供了十多种开发成熟度各异的可选服务。OpenStack 的 6 个核心服务主要担纲系统的基础架构，其余项目则负责管理控制面板、编排、裸机部署、信息传递、容器及统筹管理等操作。

* keystone：Keystone 认证所有 OpenStack 服务并对其进行授权。同时，它也是所有服务的端点目录。
* glance：Glance 可存储和检索多个位置的虚拟机磁盘镜像。
* nova：是一个完整的 OpenStack 计算资源管理和访问工具，负责处理规划、创建和删除操作。
* neutron：Neutron 能够连接其他 OpenStack 服务并连接网络。
* dashboard：web管理界面
* Swift： 是一种高度容错的对象存储服务，使用 RESTful API 来存储和检索非结构数据对象。
* Cinder 通过自助服务 API 访问持久块存储。
* Ceilometer：计费
* Heat：编排

OpenStack基本架构
![image](https://user-images.githubusercontent.com/87458342/134774086-61c2d017-78c3-4c52-ad22-2064000ffde9.png)

通过消息队列和数据库，各个组件可以相互调用，互相通信。每个项目都有各自的特性，大而全的架构并非适合每一个用户，如Glance在最早的A、B版本中并没有实际出现应用，Nova可以脱离镜像服务独立运行。当用户的云计算规模大到需要管理多种镜像时，才需要像Glance这样的组件。

OpenStack的逻辑架构

![image](https://user-images.githubusercontent.com/87458342/134774101-d9668943-34dd-4c65-9422-1704373a6337.png)

Openstack创建实例的流程
1. 通过登录界面dashboard或命令行CLI通过RESTful API向keystone获取认证信息。
2. keystone通过用户请求认证信息，并生成auth-token返回给对应的认证请求。

![image](https://user-images.githubusercontent.com/87458342/134774161-ef0996b7-4058-4678-b507-f573ab37ecb8.png)

3. 然后携带auth-token通过RESTful API向nova-api发送一个boot instance的请求。

![image](https://user-images.githubusercontent.com/87458342/134774209-5fc24180-ffb6-45a1-ac79-0a7bca7b8bb8.png)

4. nova-api接受请求后向keystone发送认证请求，查看token是否为有效用户和token。
5. keystone验证token是否有效，将结果返回给nova-api。

![image](https://user-images.githubusercontent.com/87458342/134774236-e1cc68f4-d080-417f-87f8-7a1280429763.png)

6. 通过认证后nova-api和数据库通讯，初始化新建虚拟机的数据库记录。
![image](https://user-images.githubusercontent.com/87458342/134774269-4d509101-8a94-4151-981e-820d095a2611.png)




