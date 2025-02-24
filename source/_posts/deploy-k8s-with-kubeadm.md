---
title: 用kubeadm部署k8s并检测GPU
date: 2023-03-26 01:30:14
tags: k8s kubernetes kubeadm gpu
categories: coding
copyright: true
---

> 尝试用minikube部署k8s的时候遇到一系列的问题，最终难以实现对GPU的检测,尝试分析问题，但由于minikube采用的是docker in docker的形式，所以较为复杂，改用kubeadm。minikube是单机的k8s集群，kubeadm则支持多机的k8s集群，且与minikube相比少了一层docker，尽管配置起来稍微麻烦一点，但是出了问题比较好排查。

> 此处应该是最齐全的教程，但是我部署到一半才发现这个教程。[Install Kubernetes - NVIDIA Cloud Native Technologies documentation](https://docs.nvidia.com/datacenter/cloud-native/kubernetes/install-k8s.html)

> 以下简单记录部署过程
> 系统：ubuntu16.04, 32CPU, 100GB memory, 2GPU
> NVIDIA-SMI 418.67, Driver Version: 418.67, CUDA Version: 10.1
> 本文所有操作在root用户下执行
> 最好保证内存足够大，之前采用39GB内存的系统配置起来问题一大堆

# 部署k8s

1. 安装kubeadm
  ```
  mkdir /etc/apt/keyrings
  sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg
  echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
  sudo apt-get update
  sudo apt-get install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl

  #install docker engine
  sudo apt-get install \
      ca-certificates \
      curl \
      gnupg \
      lsb-release
  mkdir -m 0755 -p /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  apt-get update
  apt-get install docker-ce 
  docker version 
  docker run hello-world
  #成功运行hello-world

  ```

2. 安装containerd
  [containerd安装官方教程](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
  ```
  #从官网下载安装包
  tar Cxzvf /usr/local containerd-1.6.2-linux-amd64.tar.gz
  bin/
  bin/containerd-shim-runc-v2
  bin/containerd-shim
  bin/ctr
  bin/containerd-shim-runc-v1
  bin/containerd
  bin/containerd-stress

  systemctl daemon-reload
  systemctl enable --now containerd
  #从官网下载runc.amd64,参照上述官方教程
  install -m 755 runc.amd64 /usr/local/sbin/runc
  #从官网下载cni安装包
  mkdir -p /opt/cni/bin
  tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
  ./
  ./macvlan
  ./static
  ./vlan
  ./portmap
  ./host-local
  ./vrf
  ./bridge
  ./tuning
  ./firewall
  ./host-device
  ./sbr
  ./loopback
  ./dhcp
  ./ptp
  ./ipvlan
  ./bandwidth
  ```

3. 配容器运行时(containerd)
  [容器运行时 官方教程](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/)
  ```
  #转发ipv4让iptables看到桥接流量
  cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
  overlay
  br_netfilter
  EOF

  sudo modprobe overlay
  sudo modprobe br_netfilter

  # 设置所需的 sysctl 参数，参数在重新启动后保持不变
  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-iptables  = 1
  net.bridge.bridge-nf-call-ip6tables = 1
  net.ipv4.ip_forward                 = 1
  EOF

  # 应用 sysctl 参数而不重新启动
  sudo sysctl --system

  lsmod | grep br_netfilter
  lsmod | grep overlay

  sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

  #先执行这个命令把配置同步过来
  containerd config default > /etc/containerd/config.toml
  #修改/etc/containerd/config.toml
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
  [plugins."io.containerd.grpc.v1.cri"]
  	sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6"

  #重启containerd
  sudo systemctl restart containerd

  #init成功，根据指引，因为是root用户，所以
  export KUBECONFIG=/etc/kubernetes/admin.conf

  ```
4. 尝试启动kubeadm
  ```
  kubeadm init --cri-socket=unix:///var/run/containerd/containerd.sock --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers

  #init成功，根据指引，因为是root用户，所以
  export KUBECONFIG=/etc/kubernetes/admin.conf
  #也可以把上面这个命令写入/etc/profile或~/.bashrc中，避免每次重开bash都要执行一次
  ```

5. 安装calico（网络插件）
  [安装calico官方教程](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)
  ```
  kubeadm reset 
  kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/containerd/containerd.sock --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers

  #确保可用内存>10G
  kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
  kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml

  #这两个命令是解除k8s不允许运行其他容器的限制
  kubectl taint nodes --all node-role.kubernetes.io/control-plane-
  kubectl taint nodes --all node-role.kubernetes.io/master-
  #也可以：kubectl taint node NodeName node-role.kubernetes.io/control-plane=:NoSchedule-

  #查看节点状态：ready
  kubectl get nodes
  #查看pods状态，全部running（需要等一会）
  kubectl get pods --all-namespaces
  ```

# 让k8s容器可以检测到GPU

> 确保GPU驱动已经正确安装，由于系统之前已经安装了所以我没有再安装

1. 安装cuda toolkit10.0(因为是ubuntu16.04，所以选择安装了这个版本)
  [下载链接](https://developer.nvidia.com/cuda-10.0-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604)

2. 安装nvidia k8s plugin插件
  [官方地址](https://github.com/NVIDIA/k8s-device-plugin#quick-start)
  ```
  distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
  curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey | sudo apt-key add -
  curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | sudo tee /etc/apt/sources.list.d/libnvidia-container.list

  sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit

  #跳过教程里configure docker阶段，因为采用containerd作为运行时，所以直接configure containerd
  #修改/etc/containerd/config.toml
  version = 2
  [plugins]
    [plugins."io.containerd.grpc.v1.cri"]
      [plugins."io.containerd.grpc.v1.cri".containerd]
        default_runtime_name = "nvidia"

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
            privileged_without_host_devices = false
            runtime_engine = ""
            runtime_root = ""
            runtime_type = "io.containerd.runc.v2"
            [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
              BinaryName = "/usr/bin/nvidia-container-runtime"

  sudo systemctl restart containerd
  kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.13.0/nvidia-device-plugin.yml


  cat <<EOF | kubectl apply -f -
  apiVersion: v1
  kind: Pod
  metadata:
    name: gpu-pod
  spec:
    restartPolicy: Never
    containers:
      - name: cuda-container
        image: nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda10.2
        resources:
          limits:
            nvidia.com/gpu: 1 # requesting 1 GPU
    tolerations:
    - key: nvidia.com/gpu
      operator: Exists
      effect: NoSchedule
  EOF

  kubectl logs gpu-pod

  #成功检测到gpu了
  kubectl describe nodes
  ```

成功啦！