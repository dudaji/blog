---
layout: post
title: "Service Account로 kubeconfig에 user 추가 및 권한 제어"
author: soonbee
categories: kubernetes
comments: true
---



kubectl를 통해 쿠버네티스 클러스터에 접근하는 사람이 꼭 관리자 한명인 것은 아니다.

관리자가 여러 명일 수도 있고 특정 namespace만을 담당하는 관리자나 읽기 권한만 있는 유저 등 다양하게 존재할 수 있다.

여러가지 방법이 있겠지만, 쿠버네티스 오브젝트 중 하나인 Service Account를 이용하면 상대적으로 간단하게 위 요구사항을 해결할 수 있다.



## 관리자

Service Account를 생성한다.

```bash
kubectl create serviceaccount hello-viewer
```



생성하고나면 해당 Service Account의 Secret이 `<service-account-name>-token-<random-string>` 라는 이름으로  자동으로 생긴다.

```
kubectl get secret
NAME                       TYPE                                  DATA   AGE
hello-viewer-token-9zmj7   kubernetes.io/service-account-token   3      5s
```



생성한 Service Account에 원하는 권한을 부여한다.

아래 명령어와 같이 [쿠버네티스에 이미 정의된 role](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles)을 사용해도 되고 직접 만든 role을 사용할 수도 있다.

```bash
kubectl create rolebinding hello-viewer-binding --clusterrole=view --serviceaccount=default:hello-viewer
```



아래 명령어로 Secret의 token을 알아낼 수 있다.

클러스터 접속을 원하는 유저에게 클러스터 정보와 함께 해당 토큰을 알려주자.

```bash
kubectl get secret hello-viewer-token-9zmj7 -o jsonpath="{.data.token}" | base64 -D
```





## 유저

클러스터에 접근하기 위해 클러스터에 대한 정보(엔드포인트 , 인증서 등)가 필요하다.

`~/.kube/config` 파일을 열어보면 아래와 같은 형식으로 이루어져 있을 것이다.

만약 해당 파일이 없다면 새로 만들면 된다.

접근하고자 하는  클러스터의 정보를 추가하자 (예시에서는 hello-cluster)



```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: LS0t...
    server: https://...
  name: ...
- cluster:
    certificate-authority-data: LS0t...
    server: https://...
  name: hello-cluster

...

contexts:
- context:
  cluster: ...
  user: ...

...

users:
- name: ...
  user:
    client-certificate-data: LS0t...
    client-key-data: LS0t...

...

current-context: ...

...
```





저장한 토큰을 이용하여 kubeconfig에 유저를 생성하자. 이름이 Service Account의 이름과 동일할 필요는 없다

```bash
kubectl config set-credentials hello-viewer --token=$TOKEN
```



위에서 정의한 클러스터와 유저를 엮어서 새로운 context를 만들고 해당 context를 사용하도록 설정한다

```bash
kubectl config set-context hello-context --cluster=hello-cluster --user=hello-viewer
```

```bash
kubectl config use-context hello-context
```



## 도커 쿠버네티스

도커에서 제공하는 쿠버네티스에서는 Service Account의 권한설정이 잘 안될 수 있다.

Cluster Role Binding 중에 이런 녀석이 하나 있는데, 이게 원인으로 보인다

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: "docker-for-desktop-binding"
subjects:
- kind: Group
  apiGroup: "rbac.authorization.k8s.io"
  name: "system:serviceaccounts"
  namespace: "kube-system"
roleRef:
  apiGroup: "rbac.authorization.k8s.io"
  kind: "ClusterRole"
  name: "cluster-amdin"
```





## 참조

https://kubernetes.io/docs/reference/access-authn-authz/rbac/

https://docs.cloud.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengaddingserviceaccttoken.htm
