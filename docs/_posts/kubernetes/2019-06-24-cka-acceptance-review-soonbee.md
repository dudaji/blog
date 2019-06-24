---
layout: post
title: "CKA(Certified Kubernetes Administrator) 합격 후기 및 팁 공유"
author: soonbee
categories: kubernetes
comments: true
---

![koonbee(256*199)](/assets/cka-acceptance-review-soonbee/koonbee.png)

안녕하세요 soonbee입니다. 

이번에 CKA에 합격하여 koonbee로 진화에 성공하였습니다.

CKA를 준비하시는 분들에게 도움이 됬으면 하는 마음으로 준비과정과 여러가지 팁들을 공유하려고 합니다.
<br/>
<br/>

## CKA란?

[Certified Kubernetes Administrator](https://www.cncf.io/certification/cka/) 의 약자로 Cloud Native Computing Foundation(CNCF)와 The Linux Foundation 에서 주관하는 Kubernetes 관련 자격증명입니다. 비슷한 녀석으로 [Certified Kubernetes Application Developer](https://www.cncf.io/certification/ckad/) (CKAD) 도 있습니다. 응시료는 300달러이고 재응시 기회를 한 번 줍니다. 그러니까 총 2번의 시험을 볼 수 있죠. **100점만점으로 74점 이상의 점수를 획득하면 합격하고 자격증명의 유효기간은 3년**입니다. (CKAD의 경우 66점 이상 합격, 유효기간 2년입니다.)

CKA의 특전(?)이라고 해야할까요? 회사에 3명 이상의 CKA 유저가 있고 [CNCF membership](https://www.cncf.io/about/join/)에 가입을 하면 그 회사는 [Kubernetes Certified Service Providers](https://www.cncf.io/certification/kcsp/) (KCSP) 인증을 신청할 수 있습니다. CNCF membership은 등급과 회사 규모에 따라 비용이 다르니 링크를 참조해주시기 바랍니다. (CKAD는 KCSP 조건에 해당하지 않습니다.)
<br/>
<br/>


## 준비과정

우선, 쿠버네티스가 아직 한국에서 크게 활성화가 되어있지 않다보니 한글로 된 자료가 거의 없습니다. 영어로 공부하셔야 합니다. 그리고 시험환경이 기본적으로 리눅스 CLI 환경이기 때문에 터미널과 기본적인 리눅스 명령어가 낯설면 조금 불리할 수도 있습니다. 그리고 네트워크쪽 기본지식이 너무 없으면 공부하는데 조금 힘들 수도 있습니다. 기초적인 수준만 알면 되기 때문에 잘 모른다고 시험을 못 볼 정도는 아닙니다.

<br/>


#### KodeKloud (추천)

[KodeKloud](https://kodekloud.com/) 에서 CKA와 CKAD 강의를 판매합니다. CKA의 경우 199달러에 판매중인데, 강의 퀄리티가 꽤 좋습니다. 애니메이션을 많이 활용해서 이해하기 쉽고 기초적인 부분부터 차근차근 알려줍니다. 그리고 **가장 좋은 점은 practice 를 제공해준다는 점인데, 실제 시험문제를 푸는 것처럼 연습문제를 풀어볼 수 있습니다**. (katacoda 환경)

단점은 영어 강의라는 점, 그리고 katacoda가 반응성이 그렇게 좋지는 않아서 좀 답답하는 것. (실제 시험에서는 반응성도 좋고 꽤 쾌적환 환경을 제공합니다.)

**여기서 꿀팁을 알려드리자면, udemy를 활용하세요.** 영어와 가격문제를 동시에 해결할 수 있습니다. kodekloud의 강의가 udemy에도 [Certified Kubernetes Administrator (CKA) with Practice Tests](Certified Kubernetes Administrator (CKA) with Practice Tests) 라는 이름으로 등록이 되어있습니다. udemy에서 구입해서 보게되면 영어자막이 나옵니다. 아쉽게도 한글자막은 없습니다. 그리고 udemy가 할인을 자주하고 꽤 많이 해주죠. udemy에서는 정가가 22만원인데 할인 받으시면 **1~2만원 근처로 구입**하실 수 있습니다. 링크 타고 들어가시면 아마 바로 할인해줄꺼에요. 할인시즌을 활용하셔도 됩니다.

저는 위 강의를 2바퀴정도 돌았습니다. 처음엔 practice 안하고 쭉 보고 두번째부터는 practice 진행하면서 (hint도 보면서) 봤구요, 그 이후로는 practice만 2~3바퀴정도 더 돌았습니다. 강의 다 보고 practice로 연습할 때는 실제 시험친다고 생각하고  (hint 절대 안보고) kubernetes document만 보면서 풀었습니다. **내용을 외우는 것보다는 해당 내용이 document에 어디에 위치하는지를 아는게 중요합니다. 그리고 practice 연습 많이 하세요.** (시험응시할 때 kubernetes 공식문서는 오픈북으로 보면서 할 수 있습니다. 자세한 내용은 아래에 있습니다. )

katacoda의 반응성 문제는 해결방법을 모르겠습니다. 네트워크 환경 좋은데서 하세요. 와이파이 느리면 딜레이 5초씩 생기기도 하고 그럽니다. ~~키보드 내리쳐서 부수지 않도록 주의하세요.~~
<br/>



#### kelseyhightower/kubernetes-the-hard-way (Github repo) (추천)

kelseyhightower 님이 작성하신 [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) 튜토리얼입니다. CKA합격이 목푤면 직접 따라할 필요까지는 없어보입니다. 저는 kodekloud 강의 다 보고 나서 2번정도 읽었습니다. 직접적으로 문제출제가 되는 영역은 아닌데 보고나면 쿠버네티스의 전체적인 이해도에 도움이 되며, Troubleshooting 문제 풀 때도 간접적으로 도움이 됩니다. 읽는 데 오래걸리지는 않으니까 보시는걸 추천해요.
<br/>



#### The Linux Foundation Training 강의

[The Linux Foundation Training](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) 에서도 강의가 있습니다. 가격은 300달러이구요, 시험에 응시할 수 있는 쿠폰과 같이 패키지로도 판매합니다. 패키지는 499달러로 같이 구매하면 100달러 싸게 구입할 수 있습니다. 저는 패키지로 구매해서 조금 봤습니다. 패키지로 구매하게 되면 메일로 쿠폰 시리얼을 주는데, 나중에 시험 결제할 때 쿠폰 적으시면 됩니다.
<br/>



#### dgkanatsios/CKAD-exercises (Github repo)

dgkanatsios 님의 [CKAD 연습문제 모음집](https://github.com/dgkanatsios/CKAD-exercises)입니다. CKAD 용이긴 하지만 CKA와 CKAD가 겹치는 영역이 많아서 겹치는 영역만 보시면 될 것 같습니다.
<br/>



#### twajr/ckad-prep-notes (Github repo)

twajr 님이 작성하신 [CKAD 요약 노트](https://github.com/twajr/ckad-prep-notes)입니다. CKAD 용이지만 이것도 마찬가지로 겹치는 영역이 많아서 그 부분만 골라서 잘 보면 될 것 같습니다. 여러가지 Tip들도 많이 적혀있어요.


<br/>
<br/>

## 시험신청 및 진행

[여기](https://training.linuxfoundation.org/wp-content/uploads/2019/05/CKA-CKAD-FAQ-May2019.pdf)를 클릭하시면 시험에 관한 Q/A 내용이 나와있습니다. 한번씩 꼭 읽어보세요!!

시험권 구매는 [CNCF 홈페이지](https://www.cncf.io/certification/cka/)에서 할 수 있습니다. 구입하면 메일이 날라와요. 거기서 cncf training portal 링크를 주는데 접속해서 회원가입하시면 시험신청이 가능합니다. **시험권 구매일 기준으로 12개월 이내로 총 2번의 응시가 가능**합니다. 시험일정을 선택하면 감독관이 배정되고 **온라인으로 시험을 진행**하게 됩니다. 시험일자는 주말, 새벽 등등 자유롭게 가능합니다만 모든 시간이 되는건 아니고 선택한 시간과 날짜를 기준으로 가능한 가장 가까운 일정을 알려줍니다. 딱 원하는 시간이 될 수도 있습니다. 시험시작 24시간 이전에는 취소하고 다시 신청할 수 있어요.

총 3시간동안 온라인으로 진행되며 총 24문제가 나옵니다. 시험지 형태의 문제는 아니고 웹터미널 환경에서 문제에서 지시하는대로 진행하시면 됩니다. 2019년 6월 기준 쿠버네티스 버전은 1.14 환경에서 진행됩니다.
<br/>



#### 접속 및 시간엄수

메일로 시험환경에 접속할 수 있는 링크를 줍니다. 15분 전부터 접속이 가능한데, 바로 들어가시는게 좋습니다. 신분증 및 주변환경 확인 등 이것저것 하는데 15분정도 걸립니다. 시간 카운트는 시험이 시작되고나서부터 진행되기 때문에 미리 접속하지 않는다고 시간적인 불이익이 있지는 않을 것 같습니다.
<br/>



#### 신분증

**여권이 필요**합니다. 그 외 가능한 목록들은 [Q/A](https://training.linuxfoundation.org/wp-content/uploads/2019/05/CKA-CKAD-FAQ-May2019.pdf)를 확인하세요.
<br/>



#### 웹캠준비

시험 시작 전에는 신분증과 시험 장소를 확인하는데 사용되고 시험을 진행하는 동안은 응시자의 얼굴을 찍습니다.
<br/>



#### 크롬 확장프로그램

시험응시 환경은 크롬만 가능합니다. 그리고 [Innovative Exams Screensharing](https://chrome.google.com/webstore/detail/innovative-exams-screensh/dkbjhjljfaagngbdhomnlcheiiangfle)를 사전에 설치해야 합니다.
<br/>



#### 컴퓨터 환경

맥북 기준으로 ⌥+⌘+ESC 눌러서 현재 작동중인 프로그램 확인하고 시험을 위한 chrome 제외하고는 종료하라고 합니다. chrome extension에 대한 제약은 없어보이는데, 저는 불편해서 Innovative Exams Screensharing 제외하고는 다 껐습니다. 특히 혹시나 Vimium 쓰시는 분들은 이거 꼭 끄세요. 웹 터미널에서 ESC키가 작동을 안해요.
<br/>



#### 접속 가능한 웹

제가 대신 전달하면 오해의 소지가 있을 수 있어서 원문 첨부합니다.

> **What resources am I allowed to access during my exam?**
> 
> Candidates may use their Chrome or Chromium browser to open one additional tab in order to
> access assets at https://kubernetes.io/docs/ and its subdomains, https://github.com/kubernetes/
> and its subdomains, or https://kubernetes.io/blog/ . No other tabs may be opened and no other
> sites may be navigated to. The allowed sites above may contain links that point to external sites.
> It is the responsibility of the candidate not to click on any links that cause them to navigate to a
> domain that is not allowed.

오픈북 시험처럼 저 3개의 도메인은 시험중에도 접속이 가능합니다. 다만 추가 탭은 1개만 가능합니다. 그 탭 내에서 왔다갔다 해요. 접속가능한 페이지에서 링크로 걸린 페이지들 또한 접속이 가능하지만, 그 페이지만 가능하고 이동한 페이지에서 다른곳으로는 이동이 안된다고 하네요. 

이것저것 설명이 복잡해보이는데, 실은 kubertetes.io/docs 이거 하나면 충분합니다. 여기서 링크 타고 외부 페이지로 갈 일도 저는 없었어요. 이렇게 docs를 활용할 수 있다보니, 위에서도 강조한 내용이지만 **내가 원하는 내용이 docs에서 어떤 키워드로 검색을 하고 어떤 페이지를 들어가야 있는지 아는게 중요**합니다. 비유하자면 도서관에서 책의 내용을 외우는게 아니라 도서관에 어디 위치에 원하는 책이 있는지를 아는게 중요한거죠.
<br/>



#### 장소

**소음이 발생되지 않는 조용한 공간**이어야 합니다. 공간 내부에는 시험 응시자만 있어야 해요. **책상 위에도 모니터, 키보드, 마우스, 충전기 등 필수적으로 필요한 것 제외하고는 다 치우라고 합니다.** (웹캠으로 확인합니다.) 종이랑 펜 안됩니다. 시험 환경에서 제공해주는 메모장만 사용 가능해요. 책상 아래도 웹캠으로 다 확인하고 캠 들고 한바퀴 돌면서 주변 환경 다 확인합니다. 시험감독관마다 case by case인것 같긴 한데, 저는 트랙패드(매직패드)도 안된다고 치우라고 해서 치웠어요. 저는 방이 좀 지저분해서 주말에 회사 회의실에서 시험봤습니다. 스터디룸같은 곳도 네트워크 상태만 양호하다면 좋을 것 같아요. https://fast.com/ko/ 에 접속하시면 네트워크 속도 간편하게 측정할 수 있습니다.
<br/>



#### 시험 환경

katacoda랑 매우 유사해요. 왼쪽에 문제가 나오고 오른쪽에 터미널화면이 있어서 거기서 작업하시면 됩니다. 상단에는 여러가지 메뉴들이 있습니다.
<br/>



#### 사용자 설정

`.vimrc` 나 `.bashrc` 설정 자유롭게 할 수 있습니다. 제가 설정한 셋팅은 아래와 같습니다.

```sh
# .vimrc
set et
set ts=2
set sw=2
set nu

# .bashrc
alias k='kubectl'
source <(k completion bash) # auto completion enable
complete -F __start_kubectl k # auto completion with command 'k'
```

`.vimrc`의 경우 tab을 2 space로 바꾸고 라인표시를 위해서 사용했구요, `.bashrc`의 경우 **kubecetl 대신 k를 사용하도록(진짜 편해요)** 그리고 [auto completion](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion)을 사용하도록 설정했습니다. 외워놨다가 시작하자마자 설정부터 하고 시작했어요.

특히 **auto completion은 정말 좋은게 오타 걱정이 없습니다.** tab을 누르면 자동완성이 되요. deployment, rolebinding 등 길이가 긴 리소스의 이름도 가능하고 create, delete 등의 command도 자동완성이 됩니다. 그리고 예를들어 pod 중 이름이 pod-ahrddss-11245 같은게 있으면 치다가 오타나거나 copy & paste 해야하는 번거로움이 있습니다. 이러한 오프젝트들의 이름도 자동완성이 가능합니다. bash completion이 설치되어 있어야 설정이 가능한데, 기본적으로 설치되어있어서 설치해야하는 수고로움은 필요 없습니다.

tmux도 사용하시면 유용합니다. 다른 node로 ssh 접속할때나 화면 스플릿 용으로 유용합니다. 기본적으로 깔려있지는 않아서 `apt install tmux` 를 통해 설치해주셔야 합니다. 다만 왜그런지는 잘 모르겠는데 tmux로 새로 생성한 세션은 위에서 설정한 것(vimrc, bashrc)들이 적용되어 있지 않더라구요. 저는 처음에 엄청 당황했는데, 여러분들은 당황하지 않고 침착하게 하시면 됩니다.
<br/>



#### 그 외 주의사항

저는 문제를 읽으면 (특히 영어) 소리 내면서 읽는 습관이 있는데, 안되더군요. **문제를 읽거나 소리내면 안되고 시험보는 중간에 입을 가려서도 안됩니다.** 저는 생각할 때 턱을 괴면서 입을 손으로 가리는 편인데 의식하면서 안하려고 노력했습니다. 

마실 것 있어도 됩니다. 다만, 투명한 유리잔 같은 것만 가능해요. (감독관마다 다를 수 있음) 라벨이 있는 음료수 같은거는 절대 안됩니다. 

중간에 break time 요청 가능합니다. 저는 쭉 풀었는데, 중간에 화장실 갔다오는 거 가능한 것 같네요. 다만, 시험시간 카운트가 멈추지는 않습니다. 시간손해는 본인이 감수해야해요.


<br/>
<br/>



## 문제

#### 출제범위

- Application Lifecycle Management 8%
- Installation, Configuration & Validation 12%
- Core Concepts 19%
- Networking 11%
- Scheduling 5%
- Security 12%
- Cluster Maintenance 11%
- Logging / Monitoring 5%
- Storage 7%
- Troubleshooting 10%
<br/>



#### 문제유형

제가 개인적으로 생각하기에 크게 4가지 유형으로 나뉘는 것 같습니다.

* object get

* object create

* troubleshooting

* node join

  

오브젝트 조회같은 경우는 주관식 문제들입니다. label이 name=soonbee 인 pod들의 이름을 적으라던가 현재 클러스에서 available 상태의 노드의 개수를 적으라던가 등등의 문제들입니다. 특정 경로에 있는 txt 파일에 답안을 적는 형태입니다. 보통 배점은 1~3점 정도.

오브젝트 생성은 조건을 주고 생성하라고 합니다. 오브젝트의 이름과 사용할 이미지등을 지정하고 생성하라고 하거나 Service와 Pod를 생성해서 연결시키거나 Configmap을 생성해서 그 값을 Pod의 env로 사용하기 등등 다양한 문제들이 있습니다. 배점은 2~4점 정도.

트러블슈팅은 시간이 조금 걸립니다. 문제는 Node등 뭔가가 Fail 등 안되는데 고쳐주세요 입니다. log 등 찍어보면서 원인을 찾고 해결하면 됩니다. 이러한 유형의 문제를 풀 때 tmux 사용하시면 유용합니다. 금방 고칠때도 있고 엄청 오래걸리기도 합니다. 배점은 4~7점정도 되는 것 같습니다.

노드 조인 문제는 1문제씩 꼭 나오는 것 같네요. 배점은 7점. node를 클러스터에 새로 추가하라는 건데, 저는 그냥 스킵했습니다. 오래 걸리기도 하고 목표는 100점이 아니라 합격이었으니까요. 문제가 엄청 길어요. 보는 순간 이거구나 싶을꺼에요.

이 외에도 DNS 등의 문제도 나옵니다.
<br/>



### 축약어

알아두시면 편합니다.

```
pod : po
replicationcontroller : rc
replicaset : rs
deployment : deploy
namespace : ns
service : svc
certificatesigningrequest : csr
ingress : ing
networkpolicies : netpol
node: no
persistentvolumeclaim : pvc
persistentvolume : pv
serviceaccount : sa
daemonset : ds

--namespace : -n
--selector : -l
```
<br/>



#### 라벨링

시험환경으로 k8s, hk8s, bk8s 등등 여러 클러스터가 있는데, 대부분의 문제가 k8s 위에서 이루어집니다. 처음에 그냥 편하게 label 붙일 때 `name: nginx` 등으로 붙였는데, 나중에 다른 문제에서도 nginx 이미지를 사용하다 보니까 중복되지 않게 하려고 `app: nginx` 이렇게 붙이고 했습니다. 근데, 이게 나중에 좀 헷갈리더라구요. 다 끝나고 나니 label 붙일 때 문항번호를 적어서 `app: 8_nginx` 나 `app: nginx` `question: 8`  등으로 2개 라벨을 활용하면 좀 더 편했을텐데 라는 생각이 들었습니다.



<br/>
<br/>


## 합격

3시간 중 2시간정도 걸려서 풀었구요, 30분정도 재검토하고 30분 남기고 조기종료했습니다. 노드 조인 문제만 스킵하면 시간은 그렇게 부족하지는 않은 것 같아요.

![cka_pass_email](/assets/cka-acceptance-review-soonbee/cka_pass_email.png)

시험을 보고나면 36시간 이내로 점수가 적힌 메일이 날라옵니다. 보통 24시간 이내로 온다는데 저는 30시간 이후에 왔어요. CKA 증명서가 첨부된 메일은 별로도 옵니다. cncf training portal 에서 유효기간 내에는 증명서 다시 다운받을 수 있습니다. 증명서는 pdf 파일이고 생긴건 아래 사진처럼 생겼습니다.

![cka_certificate](/assets/cka-acceptance-review-soonbee/cka_certificate.png)



근데 뭔가 허전하죠? 이름 부분이 비어있길래 '뭐지...?' 하고 한참을 봤습니다. 드래그해서 copy & paste 하니까 '순재 권' 이라고 나오더라구요. 구글계정으로 가입을 했는데, 구글 프로필에 제 이름이 한글로 되어있어서 그게 그대로 올라갔나봐요. 알고보니까 cncf training portal에서 check list에 Verify Name 누르면 이름 등 제 개인정보 확인 및 변경이 가능하네요. 여러분들은 꼭 확인하시고 저같은 경우는 겪지 않으시길 바랍니다. (문의메일 넣어놨는데 어떻게 되려나요..ㅠㅠ)

![check_list](/assets/cka-acceptance-review-soonbee/check_list.png)


<br/>
<br/>

## 마치며

필요한 부분만 짧게 적으려고 했는데 막상 적다보니까 내용이 길어졌네요. 준비기간은 3~4주정도 걸렸습니다. 합격도 기쁘지만 이번에 준비하고 공부하면서 쿠버네티스에 대한 이해도가 높아진 것 같아서 좋네요. 지금 이 글을 보시며 CKA 준비하시는 분들도 꼭 합격하시길 응원합니다!!