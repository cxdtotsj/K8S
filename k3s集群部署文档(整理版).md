# master

1. 一键安装K3S

    ```
    # 切换至 root 用户
    sudo su

    #- 默认安装 containerd, 无 docker
    curl -sfL https://get.k3s.io | sh -

    #- 选择 docker（须先单独安装docker,如果存在墙的情况，建议使用docker）
    curl -sfL https://get.k3s.io | sh -s - --docker
    ```

2. 获取生成的 `node-token`

    `/var/lib/rancher/k3s/server/node-token`


# node

## 第一种方法，node一键安装为agent

1. 获取 `master` 的 `node-token` （在`master`上操作）

    ```
    #切换至 root 用户
    sudo su 

    #获取token
    cat /var/lib/rancher/k3s/server/node-token
    ```

2.  `node` 加入 `master` （在 `node`上操作）

    ```
    #切换至 root 用户
    sudo su

    #- 采用默认的 containerd
    curl -sfL https://get.k3s.io | K3S_URL=https://${master_ip}:6443 K3S_TOKEN=${mater token} sh -

    #- 选择 docker （须先单独安装docker）
    curl -sfL https://get.k3s.io | K3S_URL=https://${master_ip}:6443 K3S_TOKEN=${mater token} sh -s - --docker
    ```

3. 查看`k3s`服务是否已启动

    `systemctl status k3s-agent`

## 第二种方法，node 先部署为一个单独的集群，修改配置后再加入 master 集群

1. 获取 `master` 的 `node-token` （在`master`上操作）

    ```
    #切换至 root 用户
    sudo su

    #获取token
    cat /var/lib/rancher/k3s/server/node-token
    ```

2. `node` 安装为 `master`（在`node`上操作）

    ```
    #切换至 root 用户
    sudo su
    # 安装为 master
    curl -sfL https://get.k3s.io | sh -
    ```

3. 查看 systemd

    ```
    # 查看服务状态
    systemctl status k3s
    
    # 修改服务配置信息
    vim /etc/systemd/system/k3s.service
    ```

    ```
    # k3s.service 显示内容

    [Unit]
    Description=Lightweight Kubernetes
    Documentation=https://k3s.io
    After=network.target

    [Service]
    Type=notify
    EnvironmentFile=/etc/systemd/system/k3s.service.env
    ExecStartPre=-/sbin/modprobe br_netfilter
    ExecStartPre=-/sbin/modprobe overlay
    ExecStart=/usr/local/bin/k3s server
    KillMode=process
    Delegate=yes
    LimitNOFILE=infinity
    LimitNPROC=infinity
    LimitCORE=infinity
    TasksMax=infinity

    [Install]
    WantedBy=multi-user.target

    ```

4. 修改 `ExecStart` 字段

    ```
    #- 采用默认的 containerd
    /usr/local/bin/k3s agent --server https://${master_ip}:6443 --token ${node-token}
    
    #- 选择 docker（须先单独安装docker,如果存在墙的情况，建议使用docker）
    /usr/local/bin/k3s agent --server https://${master_ip}:6443 --token ${node-token} --docker
    ```

5. 重启 `k3s` 服务

    ```
    # 重启服务
    systemctl daemon-reload
    systemctl restart k3s
    ```

# 镜像问题相关

1. 登录`master`, 查看 `kube-system`

    `kubectl get pods -n kube-system`

2. 如果遇到`pod`状态一直为 Createing，可能是镜像被墙，运行命令查看

    `kubectl describe pod ${pod_name} -n kube-system`

3. 查看`pod`的 Event，如果是 `pause` 镜像拉取失败，可以在dokcer hub上拉取镜像，改标签为google的镜像(containerd自带的 `crictl`，没有 `tag` 命令，需要使用docker)

    ```
    #(1) 从docker hub 上拉取 google 镜像文件
    docker pull mirrorgooglecontainers/pause:3.1
    
    #(2) 使用 docker tag 更改标签信息 
    docker tag docker.io/mirrorgooglecontainers/pause:3.1  k8s.gcr.io/pause:3.1
    ```

4. 删除状态为 Createing 的`pod`,会自动使用本地镜像创建系统`pod`

    `kubectl delete pod ${pod_name} -n kube-system`
