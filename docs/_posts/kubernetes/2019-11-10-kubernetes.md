---
layout: post
title: Kubernetes
author: bubuta
categories: kubernetes
comments: true
---

[15 years of experience of running production workloads at Google](https://queue.acm.org/detail.cfm?id=2898444)  
https://kubernetes.io/ 에 접속하면 볼 수 있는 문구 입니다.

### Run anywhere

Kubernetes is open source giving you the freedom to take advantage of on-premises, hybrid, or public cloud infrastructure, letting you effortlessly move workloads to where it matters to you.  

가져온 문구입니다. kuberntes에 배포할 수 있으면 infra 환경이 어디든지 쉽게 옮겨 갈 수 있습니다.  
따라서 kubernetes를 지원하면 gcp던 aws던 비슷한 환경 구성을 쉽게 할 수 있습니다.  

cloud 사업자마다 vpc, machine, service 등을 만드는 GUI, CLI 인터페이스가 달라서 한번 사용하기 시작하면
의존성이 생기고 옮기기 쉽지 않은데 그 부분에서 자유로워졌다고 볼 수 있습니다.  

하지만

1. 특수한 resource를 사용한다거나 ex.) gpu, spot instance, preemtible machine 등
2. [cluster autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) 를 사용할 경우  
cloud provider마다 편차를 제법 느낄 수 있었습니다.
