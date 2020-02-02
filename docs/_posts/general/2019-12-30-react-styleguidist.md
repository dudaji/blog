---
layout: post
title: React styleguidist
author: bubuta
categories: general
comments: true
---


그간 Full stack으로 개발하는 경험만 있었습니다. 그러다가 신규 프로젝트에서  
front, back 으로 나뉘어서 개발을 진행하게 되었습니다. 제가 참여하는 프로젝트는 아니였습니다.

1. Full stack으로 개발 할때 보다 친절한 문서가 더 많이 필요합니다. Full stack 때는 본인이 작성한 api를 본인이 사용하다보니 친절한 문서가 없었습니다.

2. 의존성이 있습니다.
아무래도 구현된 api가 있어야 test가 가능하기 때문입니다.

이 글에서는 api server에 대한 의존성 때문에 한번 작성해봅니다.  

[container components](https://medium.com/@learnreact/container-components-c0e67432e005) 이 글을 보다 보면 아래와 같은 문장이 나옵니다.
> Your component is responsible for both fetching data and presenting it. There’s nothing “wrong” with this but you miss out on a few benefits of React.

사내 프로젝트 코드를 살펴봤을 때도 몇몇 component는 fetching and presenting 두가지 일을 하는 경우를 몇 개 봤습니다.  
그런데 api server가 정상적인 작동을 하지 않으면 해당 component 개발을 못하는 경우가 발생할 수 있습니다.  
Data fetching을 못하다보니 정상적으로 browser에서 그리지 못하게 되고 코드를 변경해도 확인을 못하기 때문입니다.  
그러면 이제 차선책으로 data fetching 부분을 comment 하고 data를 hard coding 해서 작업을 진행할 수도 있겠습니다.  
하지만 한 화면안에 여러가지 component들이 있게되고 그 각각의 component들을 모두 comment해서 작업하기 쉽지 않아 보입니다.

[react-styleguidist](https://github.com/styleguidist/react-styleguidist): isolated React component development environment with a living style guide

![react-styleguidist-example-secion](https://camo.githubusercontent.com/4ee3f1442c9b5ffb568ea9ec756f093734a1d8ef/68747470733a2f2f64337676366c703535716a6171632e636c6f756466726f6e742e6e65742f6974656d732f334231323337324533763265337132553332334f2f496d616765253230323031362d30342d32302532306174253230392e31352e3234253230414d2e706e67)

그래서 이런 경우에는 react-styleguidist 를 써보면 어떨까 생각해봤습니다.  
Isolated 되어 있기 때문에 다른 환경에 영향을 덜 받고 문서화까지 할 수 있기 때문입니다.

먼저 [Presentation and Container component](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) 로 분리를 합니다.  
그리고 Presentation component 부분만 개발하면서 진도를 나가는 건 어떨지 생각해 봤습니다.  

아니면  [configureServer](https://github.com/styleguidist/react-styleguidist/blob/master/docs/Configuration.md#configureserver) 를 이용해서 endpoint를 mocking하는 것도 방법일 것 같습니다.  
default 해당 예제는 [custom endpoint](https://github.com/styleguidist/react-styleguidist/tree/master/examples/express) 입니다.

또는 test 할때 api 쏘는 부분을 mocking해서 개발해도 괜찮을 것 같습니다.  
[react testing library example](https://testing-library.com/docs/react-testing-library/example-intro) 참고
