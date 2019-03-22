# K3S离线部署

# server

## 常规部署

```
#切换至root用户
sudo su

#给k3s 二进制文件可执行权限
chomd +x k3s

#load 所需镜像
docker load --input k8s-pause.tar

#启动server服务
./k3s server --docker &

#查看服务是否已启动
./k3s kubectl get nodes
./k3s kubectl get pods -n kube-system
```

## 设置CIDR

* --cluster-cidr

    ```
    #(default: "10.42.0.0/16")
    ./k3s server --cluster-cidr ${IP} --docker &
    ```

* --service-cidr

    ```
    #(default: "10.43.0.0/16")
    ./k3s server --service-cidr ${IP} --docker &
    ```

* --cluster-dns(在 service-cidr网段下的某个 IP)

    ```
    #(default dns: "10.43.0.10")
    ./k3s server --cluster-dns ${IP} --docker &
    ```

# node1

```
#切换至root用户
sudo su

#给k3s 二进制文件可执行权限
chomd +x k3s

#load 所需镜像
docker load --input k8s-pause.tar

#启动agent服务
./k3s agent --server https://${master_ip}:6443 --token ${master-token}  --docker &

#查看node是否已加入集群(master上操作)
./k3s kubectl get nodes

```

# node2-gateway

```
#切换至root用户
sudo su

#给k3s 二进制文件可执行权限
chomd +x k3s

#load 所需镜像
docker load --input k8s-pause.tar

#启动agent服务
./k3s agent --server https://${master_ip}:6443 --token ${master-token}  --docker &

#查看node是否已加入集群(master上操作)
./k3s kubectl get nodes

#禁止 schedule pod 至该node
kubectl drain node2-gateway
#或者
kubectl cordon node2-gateway
#移除
kubectl uncordon node2-gateway

```

# 使用docker的网络配置

## docker0,flannel

```
#master 和 node 上都要操作

#查看docker.service文件路径
systemctl status docker
vim ${docker.service路径}

#添加环境变量文件地址
EnvironmentFile=/run/flannel/subnet.env

#修改 ExecStart 字段，追加参数
--bip=${FLANNEL_SUBNET} --ip-masq=${FLANNEL_IPMASQ} --mtu=${FLANNEL_MTU}

#重置并重启docker
systemctl daemon-reload
systemctl restart docker

#docker配置完成后，重启 k3s 服务
> ps -ef |grep k3s
> kill -9 ${PID}
> ./k3s server --docker &
#node上进行操作
> ./k3s agent --server https://${master_ip}:6443 --token ${master-token}

```

