# Service

1. Pod的IP在重启或者扩容时，会发生变化，如何保证一个Pod(frontend)访问另外一组的Pod(backend)，这里有一个service的概念。

2. Service的访问，分为集群内部(Pod)和集群外部(Internet)，为了满足场景，有三张类型:
    - ClusterIP: 提供一个集群内部的虚拟IP一共Pod访问
    - NodePort: 在每个Node上打开一个端口以供外部访问
    - LoadBalancer: 通过外部的负载均衡器来访问