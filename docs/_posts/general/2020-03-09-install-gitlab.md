---
layout: post
title: "GitLab 설치하기"
author: soonbee
categories: genaral
comments: true
---

# GitLab 로컬에 설치하기

총 3가지 환경에서의 설치를 다루려고 합니다.

  - [On Ubuntu](#on-premise-ubuntu-1604-lts--1804-lts)
  - [On Docker](#on-docker)
  - [On Kubernetes](#on-kubernetes)

## on-premise Ubuntu 16.04 LTS / 18.04 LTS

의존성 설치
```
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates
```
<br/>

email notification을 위한 SMTP server. 우선은 필요하지 않으므로 `No configuration` 선택합니다.
```
sudo apt-get install -y postfix
```
<br/>

gitlab 패키지 저장소 등록
```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```
<br/>

EXTERNAL_URL 은 설치할 gitlab의 도메인 주소입니다. 로컬에 설치할 것이므로 localhost를 적어줬다. 8080 포트가 사용중이라면 다른 포트를 입력합시다.
```
sudo EXTERNAL_URL="localhost:8080" apt-get install gitlab-ce
```
<br/>

curl을 이용하거나 브라우저를 통해 접속하여 잘 되었는지 확인할 수 있습니다.
```
curl localhost:8080
<html><body>You are being <a href="http://localhost/users/sign_in">redirected</a>.</body></html>
```

브라우저로 접속하면 root 계정의 password를 입력하는 화면이 나옵니다. 입력하면 로그인창으로 넘어가며 username인 root와 방금 전 입력했던 password를 통해 로그인할 수 있습니다.

<br/>

#### ubuntu on docker 주의사항

docker에서 ubuntu container를 띄워 위의 방법을 진행하는 경우 설치 도중 아래와 같은 상태에서 멈춰버립니다.
```
sudo EXTERNAL_URL="localhost:8080" apt-get install gitlab-ee
...
ruby_block[wait for redis service socket] action run // Blocking!
```

이 경우 새로운 터미널을 통해 container에 접속한 후 아래 명령어를 입력하면 설치가 진행됩니다.
```
/opt/gitlab/embedded/bin/runsvdir-start &
```

혹시 새로운 터미널을 열기 어려운 환경이거나 이미 설치를 중지시킨 상태라면 아래 명령어를 통해 다시 진행할 수 있습니다.
```
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

<br/>

## On Docker

gitlab에서 제공하는 image로 쉽게 설치할 수 있습니다.

```
sudo docker run --detach \
  --publish 8080:80 \
  --name gitlab \
  --restart always \
  gitlab/gitlab-ce:latest
```

8080 포트 외 다른 포트를 사용하고 싶다면 `--publish <사용할 포트>:80` 을 적어주세요.

<br/>

container가 내려가더라도 관련 정보를 저장하고 싶다면 volume 옵션을 추가해줍니다.

```
sudo docker run --detach \
  --publish 8080:80 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```
<br/>

curl을 이용하거나 브라우저를 통해 접속하여 잘 되었는지 확인할 수 있습니다.
```
curl localhost:8080
<html><body>You are being <a href="http://localhost/users/sign_in">redirected</a>.</body></html>
```
<br/>

## On Kubernetes

마찬가지로 gitlab에서 제공하는 image를 사용하면 편리합니다. pod가 생성된 이후로 gitlab이 실제로 running이 될 때까지 시간이 조금 걸리므로 여유로문 마음을 가지고 기다려봅시다.
<br/>

#### yaml
```
# gitlab.yaml
apiVersion: v1
kind: Pod
metadata:
  name: gitlab
  labels:
    app: gitlab
spec:
  containers:
  - name: gitlab-container
    image: gitlab/gitlab-ce:latest
---
apiVersion: v1
kind: Service
metadata:
	name: gitlab-service
spec:
	ports:
	- nodePort: 30080 # Random assignment from 30000 to 32767, if omitted
	  port: 80 # Allocate the same value as targetPort if omitted
	  targetPort: 80
	selector:
	  app: gitlab
```

```
kubectl apply -f gitlab.yaml
```
<br/>

#### command

yaml 파일 없이 명령어를 통해서도 생성할 수 있습니다.
```
kubectl create pod gitlab --image=gitlab/gitlab-ce:latest
kubectl expose pod --type=NodePort --port=80 --target-port=80 --name=gitlab-service
```
<br/>

다만 이 경우 따로 `nodePort`를 지정할 수 없어서 할당받은 포트번호를 확인해야 합니다.
```
kubectl get svc
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
gitlab-service   NodePort   10.108.75.213   <none>        80:31076/TCP     3m22s
```

31076이네요. 확인해봅시다.
```
curl localhost:31706
<html><body>You are being <a href="http://localhost:30080/users/sign_in">redirected</a>.</body></html>
```
<br/>

## 참조

https://about.gitlab.com/install/

https://docs.gitlab.com/omnibus/docker/README.html

https://gitlab.com/gitlab-org/omnibus-gitlab/issues/1504
