---
layout: post
title: "쿠버네티스 마스터 아이피가 바뀌었을때"
author: kb 
categories: kubernetes
comments: true
---


의도치 않게 쿠버네티스 Master ip가 바뀌는 케이스가 생깁니다. (예를 들어 가산에서 설치하고 거제도에서 실행할 때?) 이렇게 되면 팟들이 프로비저닝할 노드를 찾을 수 없기 때문에 정상적으로 실행되지 않습니다. 다행히 manifest에 있는 친구들은 실행되는 것을 확인 할 수 있는데요. (만약 그것도 아니라면,.... kube api-server는 떠 있어야 합니다! ) 하지만,  kube api-server도 서버 정보가 틀리기 때문에 기존의 인증 정보(~/.kube/config)로 접근할 수가 없지요. 계속 노드를 못찾는다는 메시지만 주루루룩 뜹니다... 이럴 때 어쩔수 없이 kubeadm reset을 떠올리게 되는데, 당황하지 말고 :> 한번 복구를 해봅시다.

## TL;DR

config file들의 이전 아이피를 신규 아이피로 바꾸고 이전 아이피를 가지고 있는 config map을 찾아 이전 아이피를 신규 아이피로 다 바꾼 후 컨피그 맵에 해당하는 어플리케이션의 인증서를 새로 만듭니다. 그 후에 kubelet, docker를 재시작하면 끝!

## 자, 차근차근 따라해볼까요?


### 1. /etc/kubernets 내의 설정 파일들 중 이전 아이피를 신규 아이피로 변경

```(shell)
cd /etc/kubernetes
oldip=192.168.2.200
newip=192.168.10.5

# 이전 아이피가 포함된 파일들을 조회한다.
find . -type f | xargs grep $oldip
# 이전 아이피를 신규 아이피로 바꾼다.
find . -type f | xargs sed -i "s/$oldip/$newip/"
# 정상적으로 다 바뀌었는지 확인.
find . -type f | xargs grep $newip
```

### 2. 혹시 모르니 일단 인증서를 백업해둡니다.
```(shell)
mkdir ~/k8s-old-pki
cp -Rvf /etc/kubernetes/pki/* ~/k8s-old-pki
```

### 3. /etc/kubernetes/pki에 이전 아이피를 가지고 있는 인증서가 있는지 확인.
```(shell)
cd /etc/kubernetes/pki
for f in $(find -name "*.crt"); do 
  openssl x509 -in $f -text -noout > $f.txt;
done
grep -Rl $oldip .
# 확인 되었다면 삭제 할까요? 어떤건지 알면 되니까요.
for f in $(find -name "*.crt"); do rm $f.txt; done
```

### 4. kubectl에서 --server 값으로 사용할 신규 아이피를 kubernetes 란 호스트명으로 정의
```(shell)
vi /etc/hosts
(newip) kubernetes
```

### 5. kube-system내 configmap에서 이전 아이피를 신규 아이피로 수정.
```(shell)
# configmap 파일 리스트를 조회한후, 
configmaps=$(kubectl --server=https://kubernetes:6443 -n kube-system get cm -o name | \
  awk '{print $1}' | \
  cut -d '/' -f 2)

# 일단 yaml파일로 다 생성해봅니다.
dir=$(mktemp -d)
for cf in $configmaps; do
  kubectl --server=https://kubernetes:6443 -n kube-system get cm $cf -o yaml > $dir/$cf.yaml
done

# 어떤 친구들을 수정해야 하는지 확인합니다.
grep -Hn $dir/* -e $oldip

# 수정. 여기에 해당되는 친구들은 apiserver와 etcd-server 입니다.(물론 Case by case)
kubectl --server=https://kubernetes:6443 -n kube-system edit cm kubeadm-config
kubectl --server=https://kubernetes:6443 -n kube-system edit cm kube-proxy

```
### 6. 이전 단계에서 이전 아이피를 가진 친구들의 인증서를 삭제 후 재발급합니다. 
```(shell)
# kubeadmin init phase certs {해당 어플리케이션명}
rm apiserver.crt apiserver.key
kubeadm init phase certs apiserver

rm etcd/peer.crt etcd/peer.key
kubeadm init phase certs etcd-peer

rm etcd/server.crt etcd/server.key
kubeadm init phase certs etcd-server
```

### 7. 자 kubelet과 docker를 리스타트하여 변경사항을 반영해봅시다. 처음엔 error가 한참 나겠지만 곧 복구를 하니 당황하지 말고 기다리시면 됩니다!
```(shell)
sudo systemctl restart kubelet
sudo systemctl restart docker
```

### 8. 새로운 admin.conf를 내 계정으로 이동합니다.
```(shell)
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

### 9. 마지막으로 kube-public namespace에 있는 configmap중 cluster-info에서 아이피를 새로운 아이피로 수정하면 됩니다.

### 10. 끝!


클러스터 복구만 시키는 것이기 때문에 이전 아이피를 가지고 있는 다른 서비스들은 별도의 복구 과정을 거쳐야 합니다.
