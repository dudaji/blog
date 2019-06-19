---
layout: post
title: "대학생의 KubeCon Europe 2019 후기 - day 2"
author: 최원영
categories: kubernetes
comments: true
---

이틀차부터 본격적인 KubeCon의 시작이었습니다. 아침 7시에 일어나 지하철을 타고 KubeCon 장소까지 가면서 저와 같은 KubeCon 이름표를 달고 가는 사람들을 많이 볼 수 있었습니다. 다들 아침부터 원하는 세션을 듣기 위해 부지런한 모습이었습니다. Keynote 장소에 들어서서는 왠만한 콘서트를 뛰어넘는 규모에 놀랐습니다. 생각보다 사람들이 일찍부터 많이 와있어서 앞자리는 구경도 힘들 정도였습니다.

![](/assets/2019-kubecon-europe-day2/fira_gran_via.png)
<center>< KubeCon이 열린 Fira Gran Via ></center>
<br/>

## Keynote

KubeCon을 주최하는 CNCF(Cloud Native Computing Foundation)의 여러 관계자 분들 그리고 스폰서 기업들의 엔지니어들의 키노트를 시작으로 행사가 시작되었습니다. 많은 키노트 중에도 CNCF의 디렉터인 Dan Kohn님이 말한 'Stitching together'라는 말이 가장 인상깊었습니다. 간단히 요약하자면, 현대 컴퓨터공학에서 최신 트렌드의 기술들은 **완전히 새로 개발된 것이 아니라 기존의 기술들을 엮은(stitch) 것**이고 Kubernetes 또한 마찬가지라는 말이었습니다. 저 또한 최근의 기술 발전은 아키텍쳐나 패턴의 변화로 인해 이루어졌다고 생각해서 더욱 공감이 되었습니다.

![](/assets/2019-kubecon-europe-day2/keynote.png)
<center>< 사람들로 붐볐던 Keynote 현장 ></center>
<br/>

keynote가 끝난 뒤에는 참가자들이 각자 원하는 세션이 열리는 장소로 가서 자유롭게 원하는 세션을 듣는 방식으로 일정이 진행되었습니다. 여러 세션을 들었고 그 중 흥미로웠던 두 세션에 대해 설명드리고자 합니다.

## Edge computing과 KubeEdge

최근에 관련되서 공부를 한 경험이 있어, Edge Computing에 관심이 많았습니다. 기존의 클라우드 컴퓨팅은 스마트폰, 태블릿 등 단말 기기의 부족한 컴퓨팅 성능을 대체하기 위해 단말 기기로부터 멀리 떨어져있는 클라우드 서버에 데이터를 보내 연산수행을 맡기고 그 결과값을 단말 기기로 가져오는 구조입니다. 초기에는 이런 접근이 좋은 성능을 냈지만, 점차 클라우드 컴퓨팅을 사용하는 사람들이 많아지고 실시간 데이터 처리에 대한 요구가 커지면서 클라우드 컴퓨팅만으로는 해결이 불가능한 수준에 도달하게됐습니다. 그래서, 이런 문제점을 해결하기 위해 데이터를 단말 기기 자체에서 처리를 하거나 혹은 **단말 기기에서 가까운 Edge로 데이터를 보내서 연산을 맡기는 Edge Computing**이 등장하였습니다.

Edge computing을 구현하는 엣지와 해당 엣지를 조작하는 중앙 서버를 쿠버네티스로 구성하기 위한 프레임워크로서 KubeEdge가 개발되고 있으며, 올해 가을 정식 버전이 출시될 것이라고 합니다. 세션에서는 KubeEdge가 특별하게 가지는 장점이 두가지에 대해 강조하였는데요.

첫번째는 Edge의 자율성입니다. 데이터를 처리하는 Edge와 중앙 서버의 네트워크가 충분히 reliable하지 않은 환경에서는 둘 사이의 네트워크가 단절될 경우의 recovery가 상당히 큰 이슈입니다. 그래서, KubeEdge에서는 Edge가 중앙 서버가 연결되면 중앙서버에서 네트워크가 단절된 상황에서도 자율적으로 동작할 수 있도록 필요한 metadata들을 캐싱합니다. Edge는 이를 통해 **네트워크가 단절된 상황에서도 동작이 중지되지 않고, 단독으로 효과적으로 대처**할 수 있도록 아키텍쳐를 설계하였습니다.

두번째는 edge controller를 통한 효율적인 edge와 서버간의 통신 구현입니다. KubeEdge에서는 기존에 쿠버네티스에서 pod의 모니터링을 담당하는 kubelet을 수정하여 edge controller와 cloud hub를 구현하였습니다. edge controller는 클러스터 관리자의 명령을 받아 이를 적절한 cloud hub에 전달하는 역할을 합니다. cloud hub는 각 edge에 있는 edge hub와 1대1 대응을 이루는 컴포넌트입니다. 즉, edge의 수만큼 중앙서버에 cloud hub가 존재하는데요. 각 cloud hub는 자신과 연결된 edge hub와의 통신만을 담당하기 때문에, 다른 edge와의 통신이 서로 영향을 주지 않아 더 안정적인 통신을 보장할 수 있습니다.

KubeEdge: https://kubeedge.io



## Reproducible development: Bazel & Telepresence

Reproducible development는 운영체제로 비유하자면 thread-safe와 비슷하다고 말할 수 있을 것 같습니다. reproducible development는 동일한 프로젝트가 여러 로컬머신에서 빌드되어도 그 환경에 상관없이 안전하게 원하는 결과를 내도록 하고, 또 이 결과물을 다른 머신에서 재활용하여 그에 의존되는 프로젝트를 개발할 수 있는 것을 의미합니다. 이 세션에서는 그러한 reproducible developemnt를 하기 위해 Bazel과 Telepresence를 사용한 경험을 얘기하였습니다.

Bazel은 일종의 unit build 툴입니다. 가장 큰 특징은 bazel workspace라는 개념이 있어, 한 프로젝트를 workspace 단위로 나누어 빌드할 수 있는 것입니다. 각 workspace를 빌드하면, 해당 결과물이 binary 파일로 생성되는데, 이를 다른 workspace 또는 심지어 다른 로컬 머신에서도 그대로 사용가능합니다. 또한, workspace간의 의존성을 설정할 수 있어, 상위 workspace를 빌드할 경우, 하위 workspace를 자동으로 빌드하거나 이미 빌드된 결과물을 그대로 사용가능합니다. 이런 재사용성을 통해 **불필요한 빌드과정을 줄이고 효율적으로 프로젝트를 테스트**할 수 있습니다.

Bazel: https://www.bazel.build

Telepresenece는 쿠버네티스를 이용한 서비스를 로컬에서 테스트하기 위한 라이브러리입니다. 로컬머신에서 프로젝트가 정상적으로 실행되더라도, 운영 환경과는 설정이 동일한 것이 아니어서 운영 환경에서는 에러가 나는 경우가 있습니다. 이런 일을 막기 위해 항상 운영환경과 로컬머신의 설정을 맞출 수도 있지만 상당히 비효율적이고 복잡합니다. 그래서, Telepresence는 쿠버네티스 운영환경에서 네트워크 통신, 환경변수 등의 설정을 로컬머신으로 전해줌으로서 그 문제를 해결했습니다. Telepresence를 실행시키면 쿠버네티스 운영환경에 해당 설정들을 로컬머신으로 전해주는 pod를 생성합니다. 이 pod를 통해 로컬머신의 환경을 임시로 쿠버네티스 운영환경과 동일하게 만들수 있으며, 따라서 **로컬에서도 운영환경과 동일한 테스트가 가능**합니다.

Telepresence: https://www.telepresence.io


