## 本文链接：https://mrlch.cn/archives/1572

## 文件包含如下：
- kubevirt  v0.58.0
- HostPath 
- CDI  v1.55.2
- virtctl可执行文件  v0.58.0

## 安装流程

### 节点安装libvirt、qemu
```
# Ubuntu
$ apt install -y qemu-kvm libvirt-bin bridge-utils virt-manager

# CentOS
$ yum install -y qemu-kvm libvirt virt-install bridge-utils
```

### 检查节点是否支持虚拟化
```
[root@VM-4-27-centos ~]# virt-host-validate qemu
  QEMU: Checking for hardware virtualization                                 : PASS
  QEMU: Checking if device /dev/kvm exists                                   : PASS
  QEMU: Checking if device /dev/kvm is accessible                            : PASS
  QEMU: Checking if device /dev/vhost-net exists                             : PASS
  QEMU: Checking if device /dev/net/tun exists                               : PASS
  QEMU: Checking for cgroup 'cpu' controller support                         : PASS
  QEMU: Checking for cgroup 'cpuacct' controller support                     : PASS
  QEMU: Checking for cgroup 'cpuset' controller support                      : PASS
  QEMU: Checking for cgroup 'memory' controller support                      : PASS
  QEMU: Checking for cgroup 'devices' controller support                     : PASS
  QEMU: Checking for cgroup 'blkio' controller support                       : PASS
  QEMU: Checking for device assignment IOMMU support                         : PASS
  QEMU: Checking if IOMMU is enabled by kernel                               : WARN (IOMMU appears to be disabled in kernel. Add intel_iommu=on to kernel cmdline arguments)
  QEMU: Checking for secure guest support                                    : WARN (Unknown if this platform has Secure Guest support)

```

### 安装kubevirt

项目地址： https://github.com/kubevirt/kubevirt

```
kubectl apply -f kubevirt
```
或使用官方文件进行安装
```
$ export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
$ kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
$ kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml

```

### 安装CDI

Containerized Data Importer（CDI）项是Kubernetes的持续存储管理附加组件。它的主要目标是提供一种声明性的方式来在kubevirt VMS上构建虚拟机磁盘
项目地址：https://github.com/kubevirt/containerized-data-importer

```
kubectl apply -f cdi
```
或使用官方文件进行安装
```
$ export VERSION=$(curl -s https://github.com/kubevirt/containerized-data-importer/releases/latest | grep -o "v[0-9]\.[0-9]*\.[0-9]*")
$ kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml
$ kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml
```

### 安装HostPath

```
kubectl apply -f hostpath/cert-manager.yaml
kubectl create ns hostpath-provisioner
kubectl apply -f hostpath/operator.yml  -n hostpath-provisioner
kubectl apply -f hostpath/webhook.yml  -n hostpath-provisioner
kubectl apply -f hostpath/host
```
或使用官方文件进行安装
```
# hostpath provisioner operator 依赖于 cert manager 提供鉴权能力
$ kubectl create -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.1/cert-manager.yaml

# 创建 hostpah-provisioner namespace
$ kubectl create -f https://raw.githubusercontent.com/kubevirt/hostpath-provisioner-operator/main/deploy/namespace.yaml

# 部署 operator
$ kubectl create -f https://raw.githubusercontent.com/kubevirt/hostpath-provisioner-operator/main/deploy/operator.yaml -n hostpath-provisioner
$ kubectl create -f https://raw.githubusercontent.com/kubevirt/hostpath-provisioner-operator/main/deploy/webhook.yaml
```
注意修改cr.yml里面的hostpath路径

## 镜像列表

kubevirt
```
quay.io/kubevirt/virt-api:v0.58.0
quay.io/kubevirt/virt-controller:v0.58.0
quay.io/kubevirt/virt-handler:v0.58.0
quay.io/kubevirt/virt-launcher:v0.58.0
quay.io/kubevirt/virt-operator:v0.58.0
quay.io/samblade/virtvnc:v0.1
```

hostpath

```
quay.io/kubevirt/hostpath-csi-driver:latest
k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.2.0
k8s.gcr.io/sig-storage/livenessprobe:v2.3.0
k8s.gcr.io/sig-storage/csi-provisioner:v2.2.1
quay.io/kubevirt/hostpath-provisioner-operator:latest
quay.io/jetstack/cert-manager-controller:v1.7.1
quay.io/jetstack/cert-manager-cainjector:v1.7.1
quay.io/jetstack/cert-manager-webhook:v1.7.1
```

cdi

```
quay.io/kubevirt/cdi-apiserver:v1.55.2
quay.io/kubevirt/cdi-controller:v1.55.2
quay.io/kubevirt/cdi-operator:v1.55.2
quay.io/kubevirt/cdi-uploadproxy:v1.55.2
```