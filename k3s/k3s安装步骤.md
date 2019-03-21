
# 安装 K3S 步骤（master和node都可以作为pod部署的点）

## 安装步骤

1. 在 master 上安装

    ```
    # master一键安装
    # (1) 默认为 containerd (不建议)
    curl -sfL https://get.k3s.io | sh -

    # (2) 选择 docker, 须先要安装 docker (建议，因为被墙的原因)
    curl -sfL https://get.k3s.io | sh -s - --docker


    --------------------------------------------------------

    # node 一键安装
    # 第一种方法:
    # (1) 安装成 agent
    # 默认为 containerd
    curl -sfL https://get.k3s.io | K3S_URL=https://${master_ip}:6443 K3S_TOKEN=${master token} sh -
    # 建议采用 docker 的方式
    curl -sfL https://get.k3s.io | K3S_URL=https://${master_ip}:6443 K3S_TOKEN=${master token} sh -s - --docker
    # (2) 查看 systemd
    systemctl status k3s-agent
    # (3) 打开 .service 地址，查看 env, 能看到有参数变量


    # 第二种方法:
    # (1) 安装成 master
    curl -sfL https://get.k3s.io | sh -
    # (2) 查看 systemd
    systemctl status k3s
    # (3) 打开 .service 路径地址
    cat /etc/systemd/system/k3s.service
    # (4) 修改 ExecStart 字段, 把 server 更改为 agent
    # 默认为 containerd
    /usr/local/bin/k3s agent --server https://${master_ip}:6443 --token ${NODE_TOKEN}
    # 建议采用 docker 的方式
    /usr/local/bin/k3s agent --server https://${master_ip}:6443 --token ${NODE_TOKEN} --docker
    # (5) 重置 system 文件
    systemctl daemon-reload
    # (6) 重启 k3s
    systemctl restart k3s

    ------------------------------------------------------

    # 回到 master 上操作

    # 会遇到 images 被墙的情况，通过查看日志，获取 images 的版本
    k3s kubectl get pods -n kube-system

    kubectl describe pod ${pod_name} -n kube-system

    # 如果默认使用的是 containerd ，则需要使用自带的 crictl 命令去拉取镜像，查看 containerd 默认的镜像仓库(不建议使用自带的 containerd)
    sudo crictl info

    # 查看到默认使用的是 registry-1.docker.io, 则直接拉取镜像
    # 从docker hub 上拉取 google 镜像文件，再使用 docker tag 更改标签信息 
    docker pull mirrorgooglecontainers/pause:3.1

    docker tag docker.io/mirrorgooglecontainers/pause:3.1  k8s.gcr.io/pause:3.1
    ```


2. 在 node 上安装(不建议此方法)

    ```
    # 下载二进制文件, 给执行权限
    sudo chomd +x ./k3s

    # 添加 node 至 master 集群
    sudo ./k3s agent --server https://${master_ip}:6443 --token ${master token}

    ```


## pod的命令

```
# 系统自带的namespaces
kubectl get pods -n kube-system

# 一般根据业务划分 namespaces, 可以由 .yaml 文件去指定 namespaces, 如果不知道, 则默认 default



# 查看所有的 namespaces
kubectl get namespaces

# 查看所有的pods所对应的 namespces
kubectl get pods --namesspaces

```