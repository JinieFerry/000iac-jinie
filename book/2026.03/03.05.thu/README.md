# 0304 실습 클린스탭 2

## 목표 : Hub-Spoke + NAT + Bastion + Nginx 구조를 한 번에 정상 동작시키기

## 순서 : 네트워크 → 라우팅 → TGW → NAT → EC2   
이 순서가 틀리면 apt update / ping / curl 전부 꼬인다.

## AWS HUB-SPOKE + TGW + NAT + Bastion + Nginx (Clean Version)

```
          Internet
              │
         Internet Gateway
              │
          HUB VPC
        10.0.0.0/16
      ┌──────────────┐
      │ Bastion      │
      │ 10.0.30.x    │
      └──────┬───────┘
             │
        Transit Gateway
             │
      ┌──────────────┐
      │ Nginx        │
      │ 10.10.20.x   │
      └──────────────┘
        SPOKE VPC
      10.10.0.0/16
```

1. SPOKE VPC 생성

VPC → VPC 생성

(1) VPC
+ 이름 : 0305-SPK-KR-01
+ IPv4 CIDR : 10.10.0.0/16
+ 가용 영역 수 : 2

(2) Subnet
+ 퍼블릭 서브넷 수 : 0
+ 프라이빗 서브넷 수 : 2
+ private1 : 10.10.10.0/24
+ private2 : 10.10.20.0/24
+ NAT Gateway : 없음
+ VPC Endpoint : 없음 <- 오늘은 Bastion을 Private subnet에 넣지 않을 것이므로 EC2 Instance Connect Endpoint 필요 없음
+ VPC 생성

(2-2) AWS에서 Bastion 접속 방법은 두 가지
+ Bastion을 Public subnet에 두기
```
Internet
   │
Internet Gateway
   │
Public Subnet
   │
Bastion
```

이 경우, Public IP SSH 접속이 가능해서 Endpoint가 필요없다.

+ Bastion을 Private subnet에 두기
```
  Internet
   │
AWS 내부 네트워크
   │
EC2 Instance Connect Endpoint
   │
Private subnet
   │
Bastion
```
이 경우, Public IP가 없기때문에 AWS가 Endpoint를 통해 콘솔에서 접속하게 해준다.
+ 0304의 실습이 꼬인 이유 1
```
Bastion
Subnet : private1
Public IP : Disable
```
위와 같은 설정으로 SSH 직접 접속이 불가능해서 EC2 Instance Connect Endpoint를 만들었다.

하지만 엔드포인트를 잡지 않을 경우, 아래와 같이 구조가 더 단순해진다. (Bastion → Public , Nginx → Private)
```
Internet
   │
IGW
   │
Public Subnet
   │
Bastion
   │
Private subnet
   │
Nginx
```

 
