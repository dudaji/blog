---
layout: post
title: Terraform
author: bubuta
categories: general
comments: true
---

AWS 를 사용했던 프로젝트가 있었습니다.

dev, stage, production 세 단계로 구분해서 resource를 생성했었습니다.

처음에 컨설팅 나온 업체에서는 Browser를 켜고 AWS에 접속해서 vpc 및 ec2 등 리소스등을 하나하나  
만드는 걸 보여주었습니다. dev환경에서 보여준 것을 stage, production 에서 따라 했습니다.  
그러다가 똑같은 작업을 반복하고 있는 것 같아서 aws [cloud formation](https://aws.amazon.com/cloudformation/?c=15&pt=4)을 이용해서 자동화 했습니다.   

몇일 지나고 컨설팅을 다시 받던 중에 컨설턴트 분이 [terraform](https://www.terraform.io/)을 이용하는 것을 보게 되어서 그 때 처음 접하게 되었습니다.  

terraform은 아래와 같이 infrastructure configuration를 code로 표현하여 관리 할 수 있습니다.

```terraform
rovider "aws" {
  profile    = "default"
  region     = "us-east-1"
}

provider "datadog" {
  api_key = var.datadog_api_key
  app_key = var.datadog_app_key
}

# Create a new AWS Instance
resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```

해당 configuration을 적용하면 current state와 비교해서 사용자가 정의한 대로 바꿉니다.

kubernetes에서 current state 를 desired state 로 변경하는 것과 비슷하기도 합니다.

![](https://theithollow.com/wp-content/uploads/2019/09/image.png)  
[출처 - https://theithollow.com/wp-content/uploads/2019/09/image.png]  

또한 제한적이지만 조건문과 반복문도 사용할 수 있고 모듈화도 가능합니다.
