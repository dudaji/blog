---
layout: post
title: "kubernetes에서 rbac로 service account 권한설정하기"
author: soonbee
categories: kubernetes
comments: true
---

### 시나리오

![auth-scenario](/assets/k8s-authorization-of-sa-with-rbac/auth-scenario.png)

admin : 모든 권한을 가짐

department-leader : namespace team-a와 team-b에 대한 권한을 가짐

team-a-user : namespace team-a에 대한 권한을 가짐, namespace team-b에 권한 없음

team-b-user : namespace team-b에 대한 권한을 가짐, namespace team-a에 권한 없음



### 쿠버네티스 인증방법

쿠버네티스에서 인증할 수 있는 방법들은 token, proxy, webhook, ID/PW, OAuth2 등 여러가지가 있다. 자세한 내용은 [여기](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#authentication-strategies) 에 나와있다. 쿠버네티스에서는 service account token 인증과 더불어 한 가지 이상의 유저 인증 방식을 사용할 것을 권장한다. 이 글에서는 service account(이하 SA) 인증 방식을 통해 kubernetes api 호출을 테스트하고자 한다.



### 네임스페이스 생성

```
# ns-team-a-create.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
```

```
# ns-team-b-create.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-b
```

```
kubectl create -f ns-team-a-create.yaml
```

```
kubectl create -f ns-team-b-create.yaml
```



### Service Account 생성

```
# sa-team-a-create.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-team-a
  namespace: team-b
```

```
# sa-team-b-create.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-team-b
  namespace: team-b
```

```
kubectl create -f sa-team-a-create.yaml
```

```
kubectl create -f sa-team-b-create.yaml
```



### Role & Role Binding

Role은 권한에 대한 것들을 정의한다. 어디에 접근이 가능하며 어떤 리소스에 어떠한 작업들이 가능한지가 정의된다.

RoleBinding은 Role과 User를 이어주는 역할을 한다. 즉, User에게 권한을 부여하되 그 권한에 대한 정보는 Role에 의해서 정해지는 것이다.

![role-binding](/assets/k8s-authorization-of-sa-with-rbac/role-binding.png)

Role은 custom하게 직접 정의할 수도 있고 미리 정의된 권한을 사용할 수도 있다. 미리 정의된 권한은 ClusterRole에 cluster-admin, admin, edit, view가 있으며 binding하는 종류(Rolebinding, ClusterRolebinding)에 따라 권한접근범위가 namespace인지 cluster 전체인지가 결정된다.

![default-cluster-role](/assets/k8s-authorization-of-sa-with-rbac/default-cluster-role.png)

출처: <https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles>



```
# custom-role-binding.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: custom-role
  namespace: team-a
rules:
  apiGroup: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metatdata:
  name: custom-rolebinding
  namespace: team-a
subjects:
  kind: ServiceAccount
  name: sa-team-a
  namespace: team-a
roleRef:
  kind: Role
  name: custom-role
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl create -f custom-role-binding.yaml
```



```
# default-role-binding.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: default-rolebinding
  namespace: team-b
subjects:
- kind: ServiceAccount
  name: sa-team-b
  namespace: team-b
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl create -f default-role-binding.yaml
```



### Pod 생성

```
# pod-team-a-create.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-team-a
  namespace: team-a
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 8080
```

```
# pod-team-b-create.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-team-b
  namespace: team-b
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 8080
```

```
kubectl create -f pod-team-a-create.yaml
```

```
kubectl create -f pod-team-b-create.yaml
```



### API 테스트

api server ip 확인

```
kubectl config view | grep server | cut -f 2- -d ":" | tr -d " " # display list of your api server
APISERVER=<your-k8s-api-server>
```



sa-team-a의 token 확인

```
NAMESPACE=team-a
SA=sa-team-a
TOKEN=$(kubectl -n $NAMESPACE describe secret $(kubectl get -n $NAMESPACE secrets | grep $SA | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d ' ')
```



성공

```
curl -X GET $APISERVER/api/v1/namespaces/$NAMESPACE/pods/ \
  -H "Authorization: Bearer $TOKEN" --insecure
```



sa-team-b의 token 확인

```
NAMESPACE=team-b
SA=sa-team-b
TOKEN=$(kubectl -n $NAMESPACE describe secret $(kubectl get -n $NAMESPACE secrets | grep $SA | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d ' ')
```



성공

```
curl -X GET $APISERVER/api/v1/namespaces/$NAMESPACE/pods/ \
  -H "Authorization: Bearer $TOKEN" --insecure
```


sa-team-b의 team-a 접근: Forbidden 403

```
NAMESPACE=team-a
curl -X GET $APISERVER/api/v1/namespaces/$NAMESPACE/pods/ \
  -H "Authorization: Bearer $TOKEN" --insecure
```





### department-leader와 admin 생성

leader는 namespace team-a와 team-b에 대하여 권한을 가지며 admin의 경우 모든 영역에 대하여 권한을 가진다.

```
# sa-leader.yaml
apiVersion: v1
kind: ServiceAccount
meatdata:
	name: sa-leader
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: department-leader-team-a
  namespace: team-b
subjects:
- kind: ServiceAccount
  name: sa-leader
  namespace: default
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: department-leader-team-b
  namespace: team-a
subjects:
- kind: ServiceAccount
  name: sa-leader
  namespace: default
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl create -f sa-leader.yaml
```



### Admin 생성 및 권한설정

```
# sa-admin-create.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-admin
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin-all-cluster
subjects:
- kind: ServiceAccount
  name: sa-admin
  namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

```
kubetctl sa-admin-create.yaml
```
