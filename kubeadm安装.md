# kubeadm、kubelet、kubectl安装步骤

## docker版本替换

Removing the old version with :

```
sudo apt-get purge docker-ce
sudo rm -rf /var/lib/docker
```

And then reinstall as @afbjorklund said :
```
apt-get install docker-ce=18.06.0~ce~3-0~ubuntu
```

## 国内源安装 kubeadm

```
# 可省略
apt-get update && apt-get install -y apt-transport-https

# 切换至 root
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

# 切换回用户(按需)
apt-get update

apt-get install -y kubelet kubeadm kubectl
# 安装指定版本：
apt-get install kubeadm=1.11.3-00 kubectl=1.11.3-00 kubelet=1.11.3-00
```

## 设置kubelet自动启
```
sudo systemctl enable kubelet                                             
sudo systemctl restart kubelet
```

## 关闭swap分区

```
# 只运行该命令，只在当次关闭
swapoff -a

# swappiness参数调整，修改/etc/sysctl.d/k8s.conf添加下面一行：
vm.swappiness=0

# 修改 /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"

systemctl daemon-reload

systemctl start kubelet
```

## 拉取docker hub 上的google镜像 （部署master）

docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.11.7 &&
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.11.7 &&
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.11.7 &&
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.11.7 &&
docker pull mirrorgooglecontainers/pause:3.1 &&
docker pull mirrorgooglecontainers/etcd-amd64:3.2.18 &&
docker pull coredns/coredns:1.1.3

## 更改镜像标签，还原google镜像
docker tag docker.io/mirrorgooglecontainers/kube-proxy-amd64:v1.11.7 k8s.gcr.io/kube-proxy-amd64:v1.11.7 &&
docker tag docker.io/mirrorgooglecontainers/kube-scheduler-amd64:v1.11.7 k8s.gcr.io/kube-scheduler-amd64:v1.11.7 &&
docker tag docker.io/mirrorgooglecontainers/kube-apiserver-amd64:v1.11.7 k8s.gcr.io/kube-apiserver-amd64:v1.11.7 &&
docker tag docker.io/mirrorgooglecontainers/kube-controller-manager-amd64:v1.11.7 k8s.gcr.io/kube-controller-manager-amd64:v1.11.7 &&
docker tag docker.io/mirrorgooglecontainers/etcd-amd64:3.2.18  k8s.gcr.io/etcd-amd64:3.2.18 &&
docker tag docker.io/mirrorgooglecontainers/pause:3.1  k8s.gcr.io/pause:3.1 &&
docker tag docker.io/coredns/coredns:1.1.3  k8s.gcr.io/coredns:1.1.3




## 上面是master和worker节点都需要运行的步骤

## 初始化kubeadm，启动master
```
kubeadm init --config yaml文件
```

## master部署成功
```
kubeadm join 192.168.11.133:6443 --token aru0ap.1w4q2gsjtymx88eu --discovery-token-ca-cert-hash sha256:4f4127d4be421c17715594a0caebd74141c4ac53161ab72c9bcda73479a21125
```

- 如果忘记token
```
kubeadm token list
```

## 配置安全策略
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

## 检查master节点各系统的状态
kubectl get pods -n kube-system

## 配置master网络
kubectl apply -f https://git.io/weave-kube-1.6

## 部署可视化工具 Dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

## 拉取网络配置镜像
1. 上传weave-kube-1.6至阿里云仓库
```
docker tag weaveworks/weave-npc:2.5.1 registry.cn-hangzhou.aliyuncs.com/bear_hub/weave-npc:2.5.1 &&
docker tag weaveworks/weave-kube:2.5.1 registry.cn-hangzhou.aliyuncs.com/bear_hub/weave-kube:2.5.1
```

2. worker节点登录阿里云，下载上传的镜像
```
docker login --username=1183008543@qq.com registry.cn-hangzhou.aliyuncs.com

docker pull registry.cn-hangzhou.aliyuncs.com/bear_hub/weave-npc:2.5.1 &&
docker pull registry.cn-hangzhou.aliyuncs.com/bear_hub/weave-kube:2.5.1
```

3. 变更镜像标签为原始标签
```
docker tag registry.cn-hangzhou.aliyuncs.com/bear_hub/weave-npc:2.5.1 weaveworks/weave-npc:2.5.1  &&
docker tag registry.cn-hangzhou.aliyuncs.com/bear_hub/weave-kube:2.5.1 weaveworks/weave-kube:2.5.1

```

## Worker节点
在worker上运行master的join命令:

```
kubeadm join 192.168.11.133:6443 --token aru0ap.1w4q2gsjtymx88eu --discovery-token-ca-cert-hash sha256:4f4127d4be421c17715594a0caebd74141c4ac53161ab72c9bcda73479a21125
```


## linux关机后重启

1. 重置master

`kubeadm reset`

2. 重新初始化 master

`kubeadm init --config xxx.yaml`

3. 成功后，配置安全策略

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

4. 检查master节点各系统的状态

`kubectl get pods -n kube-system`

5. 配置master网络

`kubectl apply -f https://git.io/weave-kube-1.6`

6. 配置worker节点
