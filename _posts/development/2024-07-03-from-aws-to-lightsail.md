---
layout: single
title: "AWS EC2 프리티어 기간이 만료 됐다면 Lightsail은 어떨까요"
date: 2024-07-03 03:10:00 +0900
categories: development
---

갑자기 AWS로 부터 잔액부족 메일이 왔다. 무려 25,000원이나 결제를 하라고 한다.

![mail](/assets/images/2024-07-03/aws-mail.png)

프리티어가 끝난건 알고 있었는데 25,629원이나?? 많아야 만원정도 나갈 줄 알았는데 이게 무슨 일인가.

## 비용 청구서

![청구서1](/assets/images/2024-07-03/bill1.png){: .width-500px}

![청구서2](/assets/images/2024-07-03/bill2.png){: .width-500px}

청구서를 확인해보자.

EC2가 13.11달러\... VPC가 3.6달러\...

하나씩 찾아보자

### AWS VPC 정책 변경

올해 초부터 EC2를 사용하면 VPC가 자동으로 청구된다고 한다. 프리티어는 괜찮다고 해서 신경을 껐는데 막상 과금 금액이 부담이 된다.

### EC2 비용 계산

이건 계산이 필요하다. AWS EC2의 프리티어인 t2.micro의 시간당 요금을 보면 Linux의 경우 `0.0144 USD` 이다.

$0.0144 \times 24 \times 30 = 10.368$

대략 `10.5 USD` 가 과금되어야할 텐데 왜 13달러나 과금이 되었을까?

### EC2-other..?

![cost explorer](/assets/images/2024-07-03/cost-explorer.png)

cost explorer를 보면 `EC2-기타` 라는게 청구되어있다! 이게 대체 뭐길래 돈이 추가로 나갔을까?

필터에 서비스를 EC2-Other 그리고 사용량 유형을 하나씩 선택해가며 정확히 어떤 부분에서 과금됐는지 찾을 수 있다.

![filter](/assets/images/2024-07-03/filter-result.png)

EBS Volume 때문에 대부분의 돈이 나갔다. 이걸 보고 생각이 났는데 프리티어 선택 당시 프리티어로 공짜로 쓸 수 있다고 EBS를 프리티어 한도만큼 최대로 추가 선택했었다.

그렇다. 이렇게 해서 총 16.7 달러가 나간것이였다.

### 세금과 환율

생활의 달인도 아니고 아직도 끝이 아니다. 이 가격들은 세금이 붙기 전 가격으로 세금이 추가로 붙어야한다.

세금 포함하면 비로소 `$18.37` 이 되는 것이다. 이를 현재 환율 약 1387원에 적용하자.

$18.37 \times 1387 \simeq 25,500$

이렇게 계산이 끝났다. 가볍게 생각했던 EC2 t2.micro로 기본적으로 10달러에 VPC까지 항상 붙으니 약 15달러, 2만원 가까이 매달 지출된다는 뜻이다!

## 생각보다 많은 과금 어쩌지?

가난한 취준생인 나로썬 서버로 돈을 아예 소모하고 싶지 않지만 구동중인 서버 프로그램이 존재한다.

<p align="center" style="display:flex">
    <img src="/assets/images/2024-07-03/fitness.jpg" width="45%" />
    <img src="/assets/images/2024-07-03/fitness2.jpg" width="45%" />
</p>

fitness라고 운동 기록 어플이다. `Next.js`로 만든 프론트는 `static build`로 `github pages`를 이용해 무료 배포 중이지만, `spring boot` 서버와 `redis`, `mysql`의 도커 컨테이너들은 서버가 꼭 필요했다.

- [Frontend Github Repo](https://github.com/jagaldol/behind-fitness-client)
- [Backend Github Repo](https://github.com/jagaldol/behind-fitness-server)

내가 쓰려고 만든 프로그램이고 현재 나와 내 주변 친구 몇명이서 잘 쓰고 있기 때문에 종료할 수 없다. 그렇다고 `EC2`의 `On Demand`를 활용해 일정 시간만 켜두기엔 언제나 프로그램을 사용하고 싶으니 불가. 뭔가 더 싼 서버가 없을까 고민이 됐다.

## AWS Lightsail

이런 고민을 지인에게 상담했더니 알려준 또 다른 `AWS`의 서버다. `EC2`와 달리 `on Demand`로 가격 청구가 되지 않고 고정된 가격으로 매달 지불된다.

똑같이 SSH로 서버에 접근이 가능하고 기본으로 필요한 프로그램들을 선택해 인프라 설정이 가능하다.

### Lightsail vs EC2 가성비 비교

중요한 가격부터 확인하자.

나는 별도의 App 설치가 필요없으므로(실행 할 프로그램이 docker image로 만들어 놔서 필요없다) OS only를 보았다.

![lightsail](/assets/images/2024-07-03/lightsail-select.png)

![plan](/assets/images/2024-07-03/lightsail-plan.png)

제일 싼건 `$5` 부터!

기존에 사용하던 `t2.micro`의 성능은 아래와 같다.

- 1 vCPU
- 1 GiB 메모리

7달러만 돼도 1GB 메모리에 CPU 코어가 하나 더 추가되어 `t2.micro`보다 성능이 더 좋다. 그런데도 EC2보다 싸다!

거기에 `lightsale` 첫 사용자는 3개월 간 무료라고 하니 최대한 돈을 아끼고 싶던 나에게 최적의 선택이였다.

## Lightsail 인스턴스 생성 및 서버 옮기기

방금의 화면에서 생성 버튼을 누르면 인스턴스가 만들어진다.

### Static IP

Lightsail dashboard에서 `Networking` 탭에 가면 static IP를 부여할 수 있다. 인스턴스에 붙여두면 무료로 쓸 수 있다.

> Static IP addresses are free only while attached to an instance.
>
> You can manage five at no additional cost.

### HTTPS 포트 개방 및 도메인 설정

개인 서버 도메인으로 `server.jagaldol.com`을 사용중이다. 도메인을 사용하기에 앞서 443 포트를 열어야 한다.

생성된 `Instance`의 옵션에서 `manage`로 들어가 `Networking`을 보면 `Add Rule`이 있다.

![add rule](/assets/images/2024-07-03/lightsail-rule.png)

DNS는 `Cloudflare`를 사용 중이므로 `Cloudflare`의 DNS에서 이전 EC2 주소를 `Lightsail`의 인스턴스로 변경해줬다.

![cloudflare](/assets/images/2024-07-03/cloudflare.png)

`Nginx` 설치 및 `certbot` 설정은 주제를 너무 벗어나므로 언급하지 않겠다.

### Docker 설치

항상 그렇지만 공식 문서를 읽는 습관이 가장 좋다. 여타 블로그보다 공식문서의 설치 가이드를 따르는 걸 추천한다.

- [Docker Engine 설치 가이드](https://docs.docker.com/engine/install/ubuntu/)

영어 해석이 어려워도 공식문서는 항상 코드블럭만 대충 따라쳐도 동작하게 친절하다.

### 서버 프로그램 clone 및 구동

도중에 스왑 메모리 설정, github ssh key 설정 등을 했지만 간략하게 줄이면 `clone` 후 `docker compose`로 서버 이전 성공이다!

```sh
$ git clone git@github.com:jagaldol/behind-fitness-server.git
$ cd behind-fitness-server
$ docker compose up -d
```

## 후기

과금 금액을 보고 놀라서 급하게 알아보고 변경하게 되었지만, **AWS Lightsail** 나쁘지 않다. 사양도 괜찮고 가격도 싸서 간단한 개인용 서버로 추천이다.

## 참고

- [2024년 2월부터는 EC2를 키면 VPC 요금이 나옵니다](https://6unu.net/post/public-1710173858296)
- [What is EC2-Other? \| dev.to](https://dev.to/brianpregan/what-is-ec2-other-2oi6)
- [What does the "EC2-Other" filter in the AWS Cost Explorer dashboard mean? \| stack overflow](https://stackoverflow.com/questions/56869952/what-does-the-ec2-other-filter-in-the-aws-cost-explorer-dashboard-mean)
