---
layout: post
title: "대학생의 KubeCon Europe 2019 후기 - day 1"
author: 최원영
categories: kubernetes
comments: true
---
KubeCon Europe 2019에 Dudaji와 인연이 닿아 함께 참석한 대학생 최원영입니다. 평소인프라 플랫폼 개발에도 관심이 많았지만 자주 접할 기회가 없어 아쉬웠습니다. 그리고 항상 OS 가상화 환경에서만 개발을 해왔기 때문에 최근의 트렌드인 Docker, Kubernetes를 비롯한 컨테이너 가상화는 실제로 해볼 수 있는 경험이 없어, 책으로만 공부를 조금 해본 것이 전부였습니다. 그러던 중에 SW Maestro 과정에서 멘토와 멘티로서 프로젝트를 함께 수행했던 것이 인연이 되어서, KubeCon Europe 2019가 열리는 바르셀로나에 멘토님과 함께 가게 되었습니다.


## 예상치 못한 비행기 연착

KubeCon 전체 일정은 20일 월요일부터 23일 목요일까지였기 때문에 20일 7시에 바르셀로나에 도착하는 비행기편을 예약했습니다. 인천에서 제 시간에 출발하였지만, 경유항인 홍콩에서 현지 사정으로 인해 출발하는 모든 비행기가 2시간 이상 연착이 되는 돌발상황이 일어났습니다… 결국 저희는 움직이지도 않는 비행기 안에서 몇시간을 기다린 끝에야 늦게나마 바르셀로나로 향할 수 있었습니다.

늦게 출발한 까닭에 결국 공항에도 예상보다 2시간 이상 늦게 도착하였고, 결국 오후가 다 되서야 KubeSec이 열리는 호텔에 도착할 수 있었습니다. 다행히 늦어도 기념품은 받을 수 있었습니다 :)


![freebies_of_redhat](/assets/2019-kubecon-europe-day1/freebies_of_redhat.png)
<center>< 참가자들한테 주는 기념품인 레드햇 모자와 가방 ></center>
<br/>


## KubeSec Enterprise Summit 2019

KubeSec Enterprise Summit은 Aqua security에서 주관하는 행사입니다. 하루동안 진행되며, 보안 기술 중에서도 큰 규모의 조직에서 Kubernetes를 사용하면서 발생할 수 있는 보안 이슈들이 중점적으로 다뤄집니다. 다양한 기업들의 인프라 담당자들이 보안과 관련된 자신들의 경험을 공유하기 때문에 Kubernetes 커뮤니티 내에서 최근의 보안 기술 트렌드를 느낄 수 있는 곳이기도 했습니다.

또한, 보안 컨퍼런스라서 그런지 기업 참가자분들이 상당히 많았습니다. 그래서, 주최측에서는 참가자에게 Innovation track과 Enterprise track을 선택할 수 있도록 제공해서 참가 목적에 맞는 세션을 선택해 들을 수 있었습니다. 저는 Innovartion track을 들었고, 아래와 같은 세션들이 인상깊었습니다.


#### Getting Permission or Asking for Forgiveness (liz rice, Aqua security)

리눅스 OS에서의 파일 권한 제어에 빗대어, Kubernetes에서의 RBAC(Role Based Access Control)를 통한 권한 제어에 대해서 아주 쉽게 설명한 세션이었습니다. 리눅스 파일같은 경우에는 각각 소유자, 그룹, 제 3자로 구분되는 'user'에 대해서 read, write, execute 세가지 'verb'에 대한 권한을 설정하는 방식입니다. 하지만, 리눅스의 파일에 대응되는 Kubernetes의 resource에는 소유자나 권한에 대한 설정이 존재하지 않습니다. 그래서, 이 세션에서는 이를 RBAC를 기술한 파일을 만들어 따로 관리하는 방법을 추천하고 있습니다.

**RBAC에서는 'user'를 name으로 규정하고, 그에 대한 'verb'를 각각 기술하여 관리할 수 있습니다.** 리눅스와 대응 관계를 살펴보면 아래와 비슷하다고 할 수 있을 것 같습니다. 내용 자체도 어렵지 않은 편이었지만 강사 분이 유머를 곁들여 설명해주셔서 재미있었던 것 같습니다.


![session_by_liz_rice_1](/assets/2019-kubecon-europe-day1/session_by_liz_rice_1.png)
<center>< 리눅스 파일의 권한제어와 RBAC의 대응 ></center>
<br/>

![session_by_liz_rice_2](/assets/2019-kubecon-europe-day1/session_by_liz_rice_2.png)
<center>< 엔트로피에 빗대어 설명을 하고 있는 liz rice ></center>
<br/>


#### The Enemy Within Running untrusted code with gVisor (Ian Lewis & Fabricio Voznika, Google)

컨테이너 가상화 환경에서 좀 더 강화된 샌드박스 환경을 구성할 수 있는 gVisor 런타임(https://github.com/google/gvisor)을 소개한 세션입니다. 기존의 OS 가상화에서는 VM 각각이 별개의 machine이기 때문에, VM을 일종의 샌드박스 환경으로 구성하는 것이 가능하여 보안상 유리한 측면이 있었습니다. 이런 VM의 장점을 구현하고자, 컨테이너를 실행할 때마다 별도의 가벼운 VM을 만들고 그 안에서 컨테이너를 실행시키는 Kata container(https://katacontainers.io)와 같은 오픈소스 프로젝트도 나왔지만, VM처럼 고정된 resource를 차지하는 등 효율이 매우 떨어지기 때문에 외면당했습니다.

그래서, 구글은 gVisor라는 오픈소스 프로젝트를 시작했습니다. gVisor는 유동적으로 resource를 할당받으면서도 각각의 컨테이너에 호스트와 구별되는 kernel 기능을 제공하는 컨테이너 런타임입니다. 컨테이너에서 실행되는 애플리케이션은 Go 언어로 작성된 gVisor 자체 kernel 함수를 사용할 수 있습니다. go 언어로 아예 새로운 kernel을 만들려고한 시도 자체도 놀라웠지만 특히, 샌드박스에 대한 설명부터 들으면서 평소에 크게 간과했던 부분이라는 생각이 들었던 것 같습니다.


![session_by_fabricio_voznika](/assets/2019-kubecon-europe-day1/session_by_fabricio_voznika.png)
<center>< gVisor 아키텍쳐에 대해 설명중인 Fabricio Voznika ></center>
<br/>


이외에도 다양한 세션이 있었고, 중간중간에 참가자들간의 네트워킹 그리고 패널 토크 등이 있었습니다. 특히, Kubernetes는 상당히 낮은 레벨의 소프트웨어이고 적어도 제 주변에서는 잘 아는 사람이 없기도 했는데 이렇게 많은 사람들이 그 중에서도 보안 분야에 관심을 가지는 사람이 많다는 것이 놀라웠습니다. 보안 분야에 대해 잘 알지는 못 하지만 그 규모에 놀랐던 하루였습니다.
