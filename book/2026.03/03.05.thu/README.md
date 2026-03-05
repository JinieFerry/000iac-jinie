# 0304 실습 클린스탭 2

## 목표 : Hub-Spoke + NAT + Bastion + Nginx 구조를 한 번에 정상 동작시키기

## 순서 : 네트워크 → 라우팅 → TGW → NAT → EC2   
이 순서가 틀리면 apt update / ping / curl 전부 꼬인다.

## 원칙 : 
Bastion = Public Subnet   
Nginx = Private Subnet    
NAT = Public Subnet    

## AWS HUB-SPOKE + TGW + NAT + Bastion + Nginx (Stable Version)

```
          Internet
              │
        Internet Gateway
              │
      ┌─────────────────┐
      │ HUB VPC         │
      │ 10.0.0.0/16     │
      │                 │
      │  Bastion        │
      │  10.0.10.x      │
      │  (Public)       │
      │                 │
      │  NAT Gateway    │
      │                 │
      └───────┬─────────┘
              │
        Transit Gateway
              │
      ┌─────────────────┐
      │ SPOKE VPC       │
      │ 10.10.0.0/16    │
      │                 │
      │ Nginx           │
      │ 10.10.20.x      │
      │ (Private)       │
      └─────────────────┘
```

이 경우, Public IP SSH 접속이 가능해서 Endpoint가 필요없다.

+ Bastion을 Private subnet에 두기 : Endpoint가 필요하다. 주로 보안 높은 환경에서 사용한다. (금융, 정부망)
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
위와 같은 설정으로 SSH 직접 접속이 불가능해서 EC2 Instance Connect Endpoint를 만들었다. (Private Bastion + EC2 Instance Connect Endpoint)

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

**따라서 오늘은 Bastion을 Private subnet에 두지않고 Endpoint 없이 진행한다.**

```
HUB VPC
 ├ public subnet
 │   └ Bastion
 │
 ├ public subnet
 │   └ NAT
 │
 └ private subnet
```

```
SPK VPC
 └ private subnet
     └ nginx
```

=> SSH , TGW , NAT 테스트


 
