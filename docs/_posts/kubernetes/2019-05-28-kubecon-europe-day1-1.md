---
layout: post
title: "KubeCon Europe 2019 후기(3) - day 1 : Learn by Hacking"
author: shhong
categories: kubernetes
comments: true
---

![hack](/assets/kubecon/1/hack.jpeg)

**해킹을 해보면서 쿠버네티스를 배워보는 세션**이었습니다. 같은줄인 5명이 한팀이 되어서 사전에 준비된
취약한 쿠베 클라우드를 해킹하였습니다.

가장 재밌었던 세션이었습니다.
중간에 막히면 힌트도 주고, 힌트를 바탕으로 또 해보고, 해킹을 해보면서 어떻게 쿠베가 구성되어 있고
어떤 취약점이 존재하는지를 확인하는 시간이었습니다.
영상을 시청하려면 아래를 눌러주세요.

[영상 시청](https://www.youtube.com/watch?v=NEfwUxId1Uk&list=PLj6h78yzYM2PpmMAnvpvsnR4c27wJePh3&index=55)

큰 흐름은 다음과 같습니다.

- nmap 등을 이용해서 열려있는 포트를 확인한다. (서버 주소는 사전에 주었습니다.)
- 열려있는 포트에 shellshock으로 공격한다.
- pod에 노출된 token을 찾은 뒤, secret을 얻어오고(k get secrets) decode한다.
- ssh 접속 정보를 찾는다.
- /tmp/flag에 값을 적는다.

자료는 아래에 있습니다.

[https://github.com/calinah/learn-by-hacking-kccn/](https://github.com/calinah/learn-by-hacking-kccn/)

## 에피소드

- 같은조 팀장이 상당히 잘 리딩해줘서 재밌었습니다.
- nmap, shellshock까지는 잘했는데 그다음에 막혔습니다. /tmp/k가 curl 이었는데, 이것을 못찾았습니다. 안그래도 we need curl 이라고 계속 이야기했는데 /tmp/k가 curl이라는 좀 억지스러운 가정이 있었습니다. (앞에서 힌트를 줬던것 같기도 한데 거기서 막혔습니다.)

## 참고

- [shellshock의 원리](https://operatingsystems.tistory.com/entry/Shellshock-CVE20146271)

## 느낀점

- 왜 그렇게 **PodSecurity**를 강조하는지 몸소 느낄수 있었습니다. 기존의 웹취약점 공격에 의해서 master가 통째로 빼앗기면 안되기 때문에 Pod만 빼앗기도록 해야 한다고 이야기하는 것이 바로 이것이구나 느꼈습니다.
- 꽤 괜찮은 커리큘럼이라 사내 쇼케이스때 한번 해보면 어떨까 싶었습니다. :)
