# 深入理解StatefulSet:拓扑状态

1. 步骤
    创建 statefulSet 分两个步骤，先创建 Headless 的service，再创建 statefulSet

    - 运行一个临时的Pod

        `kubectl run -it --image busybox dns-test --restart=Never --rm /bin/sh `

2. 总结

    StatefulSet 这个控制器的主要作用之一，就是使用 Pod 模板创建 Pod 的时候，对它们进行编号，并且按照编号顺序逐一完成创建工作。而当 StatefulSet 的“控制循环”发现 Pod 的“实际状态”与“期望状态”不一致，需要新建或者删除 Pod 进行“调谐”的时候，它会严格按照这些 Pod 编号的顺序，逐一完成这些操作。

    1. 首先，StatefulSet 的控制器直接管理的是 Pod。
    2. 其次，Kubernetes 通过 Headless Service，为这些有编号的 Pod，在 DNS 服务器中生成带有同样编号的 DNS 记录
    3. 最后，StatefulSet 还为每一个 Pod 分配并创建一个同样编号的 PVC