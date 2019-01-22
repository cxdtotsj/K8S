# kubernetes笔记

## 概述

![架构图](https://github.com/cxdtotsj/K8S/blob/master/pic/k8s%E6%9E%B6%E6%9E%84%E5%9B%BE.jpg)

1. 五部分核心组件(API Server、Controller Manager、Scheduler、Kubelet、etcd)：
    - Pod: 最小单元，支持多容器网络共享，文件系统共享
    - Service: 访问Pod的代理抽象服务，用于集群的服务发现和负载均衡
    - Replication Controller: 用于伸缩Pod副本数量的组件
    - Scheduler: 集群中资源对象的调度控制器(控制Pod分配到哪个Node上)
    - Controller Manager: 集群资源对象管理同步的组件
    - etcd: 分布式键值对(k,v)存储服务，存储整个集群的状态信息
    - Kubelet: 负责维护Pod容器的生命周期(Node的管理控制器)
    - Label: 用于service及Replication Controller与Pod关联的标签

2. K8S环境中假设包含一个Master节点，若干个Node节点，一个简单的Pod工作流：
    1. 提交请求: 用户通常提交一个Yaml文件，向API Server发起请求创建一个Pod
    2. 资源状态同步: Replication组件监控数据库中的数据变化，对已投的Pod进行数量上的同步
    3. 资源分配: Scheduler会检查Etcd数据库中记录的没有被分配的Pod，将Pod分配至有运行能的Node节点中，并更新Etcd中Pod的分配情况
    4. 新建容器: kubernetes集群节点中的kubelet对Etcd数据库中Pod部署状态进行同步，目标节点上的kubelet将Pod相关Yaml文件中的spec数据递给后面的容器运行时引擎(如Docker等)，后者负责Pod容器的运行停止和更新；kubelet会通过容器运行时引擎获取Pod的状态并将信息通过API Server更新至Etcd数据亏
    5. 节点通讯: Kube-proxy负责各节点中的Pod的网络通讯，包括服务发现和负载均衡

## 各组件具体实现

![Pod工作流](https://github.com/cxdtotsj/K8S/blob/master/pic/Pod%E5%B7%A5%E4%BD%9C%E6%B5%81.jpg)

1. API Server
    1. 为整个Pod工作流提供了资源对象(Pod，Deployment，Service等)的增删改查以及用于集群管理的Rest API接口，集群管理主要包括: 认证授权，集群状态管理，数据校验等
    2. 提供集群中各组件的通信及交互的功能
    3. 提供资源配额控制入口功能（每个POD占用的CPU、内存等）
    4. 安全的访问控制机制

    ![API Server工作原理]()

    在kubernetes集群中，API Server运行在Master节点上，默认开放两个端口，分别为本地端口8080 (非认证或授权的http请求通过该端口访问API Server)和安全端口6443 (该端口用于接收https请求并且用于token文件或者客户端证书及HTTP Basic的认证，用于基于策略的授权)，kubernetes默认为不启动https安全访问控制。


    API Server负责各组件间的通信，Scheduler,Controller Manager,Kubelet 通过API Server将资源对象信息存入etcd中，当各组件需要这些数据时，又通过API Server的Rest接口去获取，以下分别说明:

    - Kubelet与API Server
        
        ![kubelet监听原理]()

    - kube-controller-manager与API Server

        Controller Manager包含许多控制器，例如Endpoint Controller、Replication Controller、Service Account Controller等, 具体会在后面Controller Manager部分说明，这些控制器通过API Server提供的接口去实时监控当前Kubernetes集群中每个资源对象的状态变化并将最新的信息保存在etcd中，当集群中发生各种故障导致系统发生变化时，各个控制器会从etcd中获取资源对象信息并尝试将系统状态修复至理想状态。
    
    - kube-Scheduler与API Server

        Scheduler通过API Server的Watch接口监听Master节点新建的Pod副本信息并检索所有符合该Pod要求的Node列表同时执行调度逻辑，成功后将Pod绑定在目标节点处
    
    由于在集群中各组件频繁对API Server进行访问，各组件采用了缓存机制来缓解请求量，各组件定时从API Server获取资源对象信息并将其保存在本地缓存中，所以组件大部分时间是通过访问缓存数据来获得信息的。

2. Controller Manager

    Controller Manager在Pod工作流中起着管理和控制整个集群的作用，主要对资源对象进行管理，当Node节点中运行的Pod对象或是Node自身发生意外或故障时，Controller Manager会及时发现并处理，以确保整个集群处于理想工作状态。

    ![manager工作原理]()