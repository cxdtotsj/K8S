# 经典PaaS，作业副本（滚动更新）及水平扩展 lesson16,lesson17

## Deployment

1. `kubectl get deployments`字段含义:

    ```
    $ kubectl get deployments
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3         3         3            3           1s

    ```
    * DESIRED：用户期望的 Pod 副本个数（spec.replicas 的值）;
    * CURRENT：当前处于 Running 状态的 Pod 的个数；
    * UP-TO-DATE：当前处于最新版本的 Pod 的个数，所谓最新版本指的是 Pod 的 Spec 部分与 Deployment 里 Pod 模板里定义的完全一致；
    * AVAILABLE：当前已经可用的 Pod 的个数，即：既是 Running 状态，又是最新版本，并且已经处于 Ready（健康检查正确）状态的 Pod 的个数。

2. Deployments控制器原理 

    * Deployments控制器 通过 ReplicaSet控制器，生成所需要的Pod数量；
    * Pods更新版本，会生成对应版本的 ReplicaSet控制器；
    * 回退版本时，可通过 `kubectl rollout undo deployment/nginx-deployment`，回退至上个版本(重新调用对应版本的ReplicaSet)；
    * 可以通过 Deployment 的 spec.revisionHistoryLimit 字段，控制所保留的 ReplicaSet控制器数量，如果为0，则不支持回退版本；
    * Deployment 控制 ReplicaSet(版本)，ReplicaSet 控制 Pod(副本数)。