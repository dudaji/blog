---
layout: post
title: GCE에 설치한 쿠버네티스 클러스터를 내 로컬에서 만지고 싶어! 
author: kb
categories: kubernetes
comments: true
---
**한줄요약내용 : 기구축된 쿠버네티스 클러스터 kube-api-server의 certificate에 추가 IP, hostname을 넣기**

GCP에서 쿠버네티스를 사용하고자 한다면 GKE를 통해서 쿠버네티스 클러스터를 프로비저닝받는 것이 보통입니다.  
하지만 GCE에서 쿠버네티스를 설치하여 사용할 경우도 존재합니다. 이럴 경우 내부 네트워크를 통해서 생성된 인스턴스(ex. 10.X.X.X)이기 때문에 외부 억세스를 위해서 외부 아이피를 할당 받아야 합니다.   

하지만 kubeadm을 통해서 init한 경우 내부 IP를 default로 certificate authority(CA)가 생성이 되기 때문에 로컬피씨에서 해당 클러스터를 kube-api-server를 사용하는 kubectl로 접근하려고 한다면 아래와 같은 메시지를 반환하며

```
> kubectl get node
Unable to connect to the server: x509: certificate is valid for 10.96.x.x, 10.148.x.x, not 35.198.x.x
```

접근할 수가 없습니다. 

그 이유는 kube-api-server용 CA는 외부 아이피와는 상관이 없기 때문이죠. :)  그래서 외부 아이피를 추가시켜주어야 합니다. 추가 SAN이 필요한 것인데요. 

default SAN은 kubernetes, kubernetes.default, kubernetes.default.svc, kubernetes.default.svc.cluster.local, 10.96.0.1, 127.0.0.1, 호스트아이피로 잡혀 있습니다. 여기에다가 추가로 외부 아이피를 추가시켜주는 것이지요. 

kubeadm init의 workflow에는 아래와 같이 나와 있습니다.
 (https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#init-workflow)

```
2. Generates a self-signed CA (or using an existing one if provided) to set up identities for each component in the cluster. 
If the user has provided their own CA cert and/or key by dropping it in the cert directory configured via --cert-dir (/etc/kubernetes/pki by default). 
The APIServer certs will have additional SAN entries for any --apiserver-cert-extra-sans arguments, lowercased if necessary.
```

init 할 경우 **--apiserver-cert-extra-sans** 란 옵션을 통해 추가 SAN을 넣을 수 있다는 것인데요. 저는 이미 init을 해버린 상황이고 또 많은 강을 건너 버린 탓에 다시 init하는 것은 무리수였습니다. 

그런데 init 에 phase라는 옵션이 있습니다. 즉,  init의 특정 단계를 수행할 수 있는 것인데요. 이 phase 중에 **kubeadm init phase certs** 가 있습니다. (https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init-phase/#cmd-phase-certs) 우리는 kube-api-server의 certs에 외부 아이피를 추가해야 하기 때문에 

```
$ kubeadm init phase certs apiserver --apiserver-cert-extra-sans x.x.x.x
```

형태로 구성해 볼 수 있습니다. 하지만 이런 imperative command는  <s>뭘 추가한지 까먹기마련입니다.</s>  관리차원에서 지양하는게 낫습니다. 그러면 configmap같은 거에 설정하면 좋을거 같은데요. 아마 kube-system에 kubeadm관련  config가 있을 겁니다.

```
$ leemyounghwan@kb-instance1:~$ kubectl get configmap -n kube-system
NAME                                           DATA   AGE
calico-config                                  4      16d
cert-manager-cainjector-leader-election        0      6d23h
cert-manager-cainjector-leader-election-core   0      6d23h
cert-manager-controller                        0      6d23h
cert-manager-kube-params-parameters            1      6d23h
coredns                                        1      16d
extension-apiserver-authentication             6      16d
kube-proxy                                     2      16d
kubeadm-config                                 2      16d
kubelet-config-1.15                            1      16d
my-release.v1                                  1      16d

$ kubectl get configmap kubeadm-config -n kube-system -o yaml
apiVersion: v1
data:
  ClusterConfiguration: |
    apiServer:
      extraArgs:
        authorization-mode: Node,RBAC
      timeoutForControlPlane: 4m0s
    apiVersion: kubeadm.k8s.io/v1beta2
    certificatesDir: /etc/kubernetes/pki
    clusterName: kubernetes
    controllerManager: {}
    dns:
      type: CoreDNS
    etcd:
      local:
        dataDir: /var/lib/etcd
    imageRepository: k8s.gcr.io
    kind: ClusterConfiguration
    kubernetesVersion: v1.15.11
    networking:
      dnsDomain: cluster.local
      podSubnet: 192.168.0.0/16
      serviceSubnet: 10.96.0.0/12
    scheduler: {}
  ClusterStatus: |
    apiEndpoints:
      kb-instance1:
        advertiseAddress: 10.x.x.x
        bindPort: 6443
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterStatus
kind: ConfigMap
...

```

 역시 있습니다. <s>짬에서 나오는 바이브로 봤을 때</s> apiServer에 뭔가 넣어주면 될거 같은데요. 그러면 kubeadm.k8s.io/v1beta2 관련 스펙만 찾으면 되니까... 공식사이트에서 kubeadm 관련 내용을 찾습니다.

https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file 에 보면

```
For more details on each field in the v1beta2 configuration you can navigate to our API reference pages.
```

란 내용이 있고... 해당 링크를 따라가면 kubeadm.k8s.io의 스펙을 찾을 수 있습니다. 
(https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2?tab=doc)

 스펙문서를 훑어가다 보면 kubeadm init할때 풀 스택? 버전의 yaml 파일 예제가 있습니다. 우리는 apiServer에 관심이 있으니 그 부분만 찾아서 보면

```
...
apiServer:
  extraArgs:
    authorization-mode: "Node,RBAC"
  extraVolumes:
  - name: "some-volume"
    hostPath: "/etc/some-path"
    mountPath: "/etc/some-pod-path"
    readOnly: false
    pathType: File
  certSANs:
  - "10.100.1.1"
  - "ec2-10-100-0-1.compute-1.amazonaws.com"
  timeoutForControlPlane: 4m0s
...
```

오호! certSANs: 가 우리가 찾는 그것인거 같습니다. 

그러면 원래 우리 것에다가 certSANs를 추가해봅시다. 먼저 configmap에서 ClusterConfiguration을 kubeadm-conf.yaml 파일로 추출하고,

```
$ kubectl get configmap kubeadm-config -n kube-system -o jsonpath='{.data.ClusterConfiguration}' > kubeadm-conf.yaml
```

certSANs 필드를 우리가 추가할려는 외부 아이피를 넣어 추가합니다.

```
...
apiServer:
  certSANs:
  - 35.198.x.x
  extraArgs:
    authorization-mode: Node,RBAC
  timeoutForControlPlane: 4m0s
...
```

자, 그러면 이제 cert를 생성할 차례인데요. 먼저 기존의 kube-api-server의 cert, key파일을 이동시킵니다. kubeadm은 certificatesDir에 설정된 곳에 해당 cert/key가 있다면 새로 만들지 않으니까요 :)

이동이 완료되면 우리가 무시한 imperative command였던 kubeadm init phase certs apiserver으로 cert/key를 재생성합니다. 옵션 중에  --config를 통해 kubeadm-conf.yaml을 입력 받습니다.

```
$ sudo kubeadm init phase certs apiserver --config kubeadm-conf.yaml
```

아마 typo만 없다면 정상적으로 생성될 것입니다.  그리고 apiserver를 재실행 시킵니다.

자 그러면 cert가 정상적으로 생성됬는지 확인해 볼까요.

```
$ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
...
            X509v3 Subject Alternative Name:
                DNS:kb-instance1, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:10.96.0.1, IP Address:10.148.x.x, IP Address:35.198.x.x
...                
```

앞서 설명한  default 친구들과 추가된 외부 아이피까지 같이 SAN 으로 등록되어 있는 것을 확인할 수 있습니다.

여기서 끝이면 좋겠지만 kubeadm을 통해 클러스터의 변경사항이 있을 때 추가한 외부 아이피도 같이 반영이 되어야 하기 때문에 kubeadm-conf.yaml 이 친구를 kubeadm-config의 ClusterConfiguration 필드에 업데이트를 해야합니다.

```
$ kubeadm config upload from-file --config kubeadm-conf.yaml
$ kubectl get configmap kubeadm-config -n kube-system -oyaml
apiVersion: v1
data:
  ClusterConfiguration: |
    apiServer:
      certSANs:
      - 35.198.x.x
      extraArgs:
        authorization-mode: Node,RBAC
      timeoutForControlPlane: 4m0s
    apiVersion: kubeadm.k8s.io/v1beta2
    certificatesDir: /etc/kubernetes/pki
...    
```

 자, 이제 대장정이 끝이 났습니다.  이제 다시 로컬에서 kubectl로 접근해봅시다.

```
> kubectl get node
NAME           STATUS   ROLES    AGE   VERSION
kb-instance1   Ready    master   16d   v1.15.5
```

성공입니다.   

물론 이 방법말고도 더 간단한 방법이 존재할 거 같긴합니다만...

