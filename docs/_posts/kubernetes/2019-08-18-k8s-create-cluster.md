---
layout: post
title: "쿠버네티스 클러스터 구성하기"
author: soonbee
categories: kubernetes
comments: true
---

![koonbee(256*199)](/assets/cka-acceptance-review-soonbee/koonbee.png)

안녕하세요 soonbee입니다. 이번에는 쿠버네티스 클러스터 구성기입니다.

저희 사내 서버는 현재 쿠버네티스를 활용하여 클러스터 구축이 되어있습니다. node 중 하나가 master 겸 worker로 구성되어 있는데, 이를 분리해야하는 니즈가 발생했습니다.

첫번째로는 보안이슈. worker node 하나가 물리적으로 master node이기도 하기 때문에 worker를 사용하는 사용자가 클러스터의 시스템적인 부분을 건드릴 가능성이 있었습니다. 사내 서버이기 때문에 외부에서의 악의적인 접속보다는 누가 실수로 잘못 건드는 일을 방지하는 목적이 좀 더 크긴 합니다.

두번째로는 안정성. 실은 이 문제가 메인 이슈입니다. 인공지능 학습등을 위해 서버를 돌리다가 좀 하드하게 돌리면 가끔 node가 뻗어버리는 상황이 발생하는데, 이 때 뻗어버리는 node가 master & worker node인 경우 서버 전체에 영향을 미칩니다. 그렇다고 조심하느라 제대로 사용 못하면 이것도 문제죠. 그래서 현재 master & worker node는 worker node로 사용하고 새로 pc를 구입하여 master node로 활용하기로 했습니다.

master로 사용한 pc는 인텔의 베어본pc NUC7CJYH (RAM-8G / SSD-240GB) 입니다.

<br/>
<br/>

## os 설치

os는 ubuntu 18.04를 사용했다. 부팅usb를 만들 때 [Etcher](https://www.balena.io/etcher/)를 사용하면 편하게 만들 수 있습니다.

혹시 한글설정이 필요하다면 [여기]([https://gabii.tistory.com/entry/Ubuntu-1804-LTS-%ED%95%9C%EA%B8%80-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EC%84%A4%EC%A0%95](https://gabii.tistory.com/entry/Ubuntu-1804-LTS-한글-설치-및-설정))가서 따라하시면 됩니다.

<br/>
<br/>

## docker 설치

쿠버네티스 공식문서에서는 도커 버전 18.06.2 를 권장하고 있습니다. 그 외 버전이 안되는건 아니므로 크게 걱정할 필요는 없어보입니다. 자세한 정보는 [쿠버네티스 공식문서](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) 를 참고해주세요.

```bash
# Install Docker CE
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
apt-get update && apt-get install apt-transport-https ca-certificates curl software-properties-common

### Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

### Add Docker apt repository.
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

## Install Docker CE.
apt-get update && apt-get install docker-ce=18.06.2~ce~3-0~ubuntu

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart docker.
systemctl daemon-reload
systemctl restart docker
```

<br/>
 
## 사전 설치 및 준비

아래 표에 적혀있는 포트들은 쿠버네티스가 사용하는 포트들이므로 비워둡시다.
<br/>

### Master Node

| Protocol | Direction | Port Range | Purpose                 | Used By              |
| :------- | :-------- | :--------- | :---------------------- | :------------------- |
| TCP      | Inbound   | 6443*      | Kubernetes API server   | All                  |
| TCP      | Inbound   | 2379-2380  | etcd server client API  | kube-apiserver, etcd |
| TCP      | Inbound   | 10250      | Kubelet API             | Self, Control plane  |
| TCP      | Inbound   | 10251      | kube-scheduler          | Self                 |
| TCP      | Inbound   | 10252      | kube-controller-manager | Self                 |

<br/>

### Worker Node

| Protocol | Direction | Port Range  | Purpose             | Used By             |
| :------- | :-------- | :---------- | :------------------ | :------------------ |
| TCP      | Inbound   | 10250       | Kubelet API         | Self, Control plane |
| TCP      | Inbound   | 30000-32767 | NodePort Services** | All                 |

<br/>

그 다음 쿠버네티스 클러스터 구성을 위해 필요한 `kubeadm`, `kubelel`, `kubectl` 을 설치해줍시다.
```sh
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```
<br/>



쿠버네티스 클러스터 구동 시 swap을 사용할 수 없습니다. 완전 불가능은 아닌데 여러가지로 복잡하므로 꺼줍시다.
```
swapoff -a
```
<br/>
<br/>


## 클러스터 구성하기 (master)

```
kubeadm init --pod-network-cidr=10.244.0.0/16
```

`--pod-network-cidr` 옵션은 후에 pod network 구성을 위한 옵션입니다. 어떤 녀석을 사용할지에 따라 옵션 여부 및 값이 달라집니다. 자세한 것은 [쿠버네티스 공식문서](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)를 확인해주세요. 여기선 Flannel을 사용해보도록 하겠습니다. 이미 init을 해버린 상태라면 `kubeadm reset` 을 통해 리셋하고 다시 하면 됩니다.

성공적으로 init이 되었다면 아래와 같은 메세지들이 출력됩니다.

```
[init] Using Kubernetes version: vX.Y.Z
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kubeadm-master localhost] and IPs [10.138.0.4 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kubeadm-master localhost] and IPs [10.138.0.4 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubeadm-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.138.0.4]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 31.501735 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-X.Y" in namespace kube-system with the configuration for the kubelets in the cluster
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "kubeadm-master" as an annotation
[mark-control-plane] Marking the node kubeadm-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node kubeadm-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: <token>
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

맨 마지막 `kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>` 부분을 어딘가에 잘 적어놓도록 합시다. 다시 확인하거나 새로 생성할 수 있긴 한데 귀찮아집니다. 그냥 줄 때 잘 적어두는게 편합니다.

Your Kubernetes master has initialized successfully! 라는 멘트 아래에 보면 config 파일을 특정 경로에 복사해두라는 말이 있습니다. 여러분이 앞으로 kubectl을 통해 명령을 내릴 때 어느 클러스터에 명령을 내릴 지 등의 설정이 저장된 파일입니다. su 모드일때와 아닐 때 `$HOME` 이 다르다는 걸 유의합시다.

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

<br/>

그 다음은 pod network 설정입니다.

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml
```

명령어 실행 후 `kubectl get pods --all-namespaces` 명령어를 치면 컨테이너들이 생성중인 것을 확인할 수 있습니다.

<br/>
<br/>


## 클러스터 구성하기(Worker)

worker 구성은 master 구성보다 간단합니다.

worker node 에서도 위와 같이 docker, kubeadm, kubelet, kubectl을 설치해줍시다.

이 후 잘 옮겨적어놨던 명령어 `kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>` 를 입력하면 됩니다. `kubeadm join` 에 사용되는 token의 유효기간은 24시간이며 혹시라도 유효기간이 의심되거나 위 명령어를 잃어버렸다면 master node에서 `kubeadm token create --print-join-command` 명령어를 통해 새로운 토큰 생성하고 필요한 명령어를 볼 수 있습니다. join이 잘 되었는지 확인하고 싶다면 master node에서 `kubectl get nodes` 를 통해 확인하시면 됩니다.

<br/>
<br/>

## 참조

https://kubernetes.io/docs/setup/production-environment/container-runtimes/

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/