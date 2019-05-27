---
layout: post
title: "k8s private network 에서 let's encrypt 인증서 발급 받기"
author: bubuta
categories: kubernetes
comments: true
---

## 배경
Docker registry에 self signed certificate를 사용할 경우 Host에 따로 인증서를 추가해주거나
insecure registry를 추가주어야 하는 번거로움이 있습니다.

## How it works
[let's encrypt challenge type](https://letsencrypt.org/docs/challenge-types/)  
dns01 challenge를 이용하면 private network에서 lets encrypt 인증서를 받을 수 있습니다.    
DNS Provider 계정 정보를 통해 도메인 소유를 증명합니다.

## 준비
1. 도메인이 미리 구매되어 있어야 합니다.
2. GCP 계정이 이미 있어야 합니다.


## name server 변경
https://www.ihee.com/322 이 문서를 참고하시면 됩니다.  
Domain을 구매한 사이트에서 네임서버를 google cloud dns로 변경합니다.

## service account 생성 및 Cert manager Cluster issuer 생성
https://knative.dev/docs/serving/using-cert-manager-on-gcp/  
위 문서를 참고하여 dns 권한을 갖고 있는 service account를 생성합니다.  
그리고 cert manager에서 Cluster issuer를 생성합니다.

> 이때 바로 server: https://acme-v02.api.letsencrypt.org/directory를 사용하지 마시고  
[lets encrypt staging environment](https://letsencrypt.org/docs/staging-environment/)  를 사용하시는게 좋습니다.  [rate limit](https://letsencrypt.org/docs/rate-limits/)이 존재하기 때문입니다.


## 인증서 자동 생성
https://docs.cert-manager.io/en/latest/tasks/issuing-certificates/ingress-shim.html

ingress 설정을 할때는 annotations에 dns 01, clouddns를 사용하는 것을 명시해 줍니다.
```
certmanager.k8s.io/acme-challenge-type: dns01
certmanager.k8s.io/acme-dns01-provider: clouddns
```
