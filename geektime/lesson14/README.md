# 深入解析Pod对象

## Pod 中几个重要字段的含义和用法

1. **NodeSelector**: 是一个供用户将 Pod 与 Node 进行绑定的字段，用法如:

    ```
    apiVersion: v1
    kind: Pod
    ...
    spec:
    nodeSelector:
    disktype: ssd

    ```
    这样的一个配置，意味着这个 Pod 永远只能运行在携带了"disktype: ssd”标签（Label）的节点上；否则，它将调度失败.

2. **NodeName**: 一旦 Pod 的这个字段被赋值，Kubernetes 项目就会被认为这个 Pod 已经经过了调度，调度的结果就是赋值的节点名字。所以，这个字段一般由调度器负责设置，但用户也可以设置它来“骗过”调度器，当然这个做法一般是在测试或者调试的时候才会用到。

3. **HostAliases**： 定义了 Pod 的 hosts 文件（比如 /etc/hosts）里的内容，用法如下：

```
apiVersion: v1
kind: Pod
...
spec:
  hostAliases:
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
...

```

4. **凡是跟容器的 Linux Namespace 相关的属性，也一定是 Pod 级别的。**

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  shareProcessNamespace: true
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    stdin: true
    tty: true

```