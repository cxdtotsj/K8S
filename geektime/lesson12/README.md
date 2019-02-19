# LessonName: 我的第一个容器化应用(nginx)

## 概述

- 部署一个nginx并更新

```
kubectl apply -f nginx-deployment.yaml   (推荐)
kubectl create -f nginx-deployment.yaml

# 修改 nginx-deployment.yaml 的内容

kubectl apply -f nginx-deployment.yaml  (推荐)
kubectl replace -f nginx-deployment.yaml

```

- 查看一个容器的详细信息

```
# 查看pod
kubectl get pods
kubectl describe pod 容器名称

# 进入容器
kubectl exec -it 容器名称 -- /bin/bash

```

- 删除pod

```
kubectl delete -f nginx-deployment.yaml
```