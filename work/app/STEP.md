# 概述

- 部署一个nginx并更新

```
kubectl apply -f nginx-deployment.yaml   (推荐)
kubectl create -f nginx-deployment.yaml

# 修改 nginx-deployment.yaml 的内容

kubectl apply -f nginx-deployment.yaml  (推荐)
kubectl replace -f nginx-deployment.yaml

```