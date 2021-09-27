# OpenStack知识体系

## 第1章：OpenStack介绍
* OpenStack定义
* OpenStack架构

## 第2章：OpenStack实验环境部署
* 安装方法与工具概述
* 实验环境安装

## 第3章：验证授权与服务编目-Keystone
* Keystone原理
  * 什么是Keystone?
  * Keystone的主要功能
  * Keystone的概念
  * 示例：Keystone与其它服务交互的流程
* 实验：
  * 启用启动服务器后，DevStack的启动
  * 通过图形界面的Horizon访问Openstack
  * 通过命令行访问Openstack
  * 通过REST API访问OpenStack
  * 管理项目、用户、角色

## 第4章：镜像服务-Glance
* 什么是Image
* Glance原理
  * Glance体系结构
  * Glace支持的镜像格式和容器
  * 镜像的属性、权限与状态
  * 制作镜像的思路
* 实验：
  * 考察现有镜像（GUI、CLI）
  * 上传新的镜像（GUI、CLI）
  * 修改镜像属性（仅能用CLI）
  * 删除镜像

## 第5章：计算服务-Nova
* Nova原理
  * Nova体系结构
  * Nova组件功能与交互流程
  * 实例类型
  * 计算节点的选择调度与Driver架构
* 实验：
  * 实例创建与控制
  * 实例的操作（GUI、CLI）
  * 启动与关闭
  * 重新启动
  * 锁定与解锁
  * 暂停与挂起
  * 大小调整
  * 废弃与取回
  * 删除

## 第6章：块存储服务-Cinder 
* 创建实例时存储的选项
* Cinder原理
  * Cinder体系结构
  * Cinder组件交互流程
  * Cinder的调度算法
  * Cinder-volume的Driver架构
* 实验：
  * 创建卷
  * 连接卷到实例
  * 分离卷
  * 扩展卷
  * 卷的快照
  * 删除卷
  * NFS Volume Provider

## 第7章：网络服务-Neutron（基础）
* Neutron原理：
  * 概述与功能
  * 基本概念与架构
  * Neutron Server分层模型
  * ML2 Core Plugin与Agent
  * Service Plugin与Agent
* 实验：
  * 配置Linuxbridge
  * 创建Local Nertwork（Linuxbridge）
  * 创建Flat Nertwork（Linuxbridge）
  * 配置DHCP Agent
  * 创建VLAN Network（Linuxbridge）
  * 创建Routing （Linuxbridge）

