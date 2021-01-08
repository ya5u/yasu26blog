---
title: "Raspberry Pi 4 で Kubernetes をセットアップする"
date: 2021-01-07T22:57:44+09:00
draft: true
tags:
- Kubernetes
- Raspberry Pi
---

[Raspberry Pi 4 でつくる ”おうち Kubernetes Cluster” シリーズ](/posts/k8s-create-cluster) の Kubernetes Cluster 構築編です

## cgroup 有効化

/boot/firmware/cmdline.txt に以下を追記して cgroup を有効化します

```
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
```

## CRI インストール

Kubernetes v1.20 から Docker が非推奨になっているので今回は containerd を選択しました

参考: [Container Runtimes#containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

まず前提となる設定を追加します

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

そして containerd をインストール

```bash
# (Install containerd)
## Set up the repository
### Install packages to allow apt to use a repository over HTTPS
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

```bash
## Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add -
```

**※ 公式では次のコマンドの2行目で [arch=amd64] となっていますが、今回は Raspberry Pi 上で構築するので [arch=arm64] とします**

```bash
## Add Docker apt repository.
sudo add-apt-repository \
    "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```

```bash
## Install containerd
sudo apt-get update && sudo apt-get install -y containerd.io
```

containerd の設定を追加

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

containerd をリスタート

```bash
sudo systemctl restart containerd
```
