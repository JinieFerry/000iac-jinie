
# 010. 콘솔 기초 실습1

## 0. 실습 목표

* EC2 시작 템플릿 생성
* Nginx 자동 설치
* Instance Name / Instance ID 출력
* Auto Scaling Group 구성
* ALB 로드밸런싱 구성
* VPC 네트워크 구조 이해

## 00. 구조 흐름
```
Internet
   │
   ▼
Internet Gateway (IGW)
   │
   ▼
Public Subnet
   │
   ├─ ALB (Application Load Balancer)
   │
   └─ NAT Gateway
          │
          ▼
      Private Subnet
          │
          ▼
   Auto Scaling Group
          │
          ▼
        EC2 (Nginx)
          │
          ▼
   index.html
   Instance Name
   Instance ID
```

## 000. 네트워크 흐름

1. 사용자 접속

```
User
↓
ALB
↓
Target Group
↓
Auto Scaling EC2
↓
Nginx index.html
```

2. 서버 인터넷 접근

```
Private EC2
↓
NAT Gateway
↓
Internet
```
---

# 0000. 아키텍처 설계
1. 구조

```
Internet
   ↓
IGW
   ↓
Public Subnet
   ↓
ALB
   ↓
Target Group
   ↓
Auto Scaling Group
   ↓
EC2 (Nginx)
```

2. Private EC2 인터넷 접근

```
Private EC2
   ↓
NAT Gateway
   ↓
Internet
```

---

# 1. VPC 생성
## 1) 0305-vpc-01 생성
```
VPC
0305-vpc-01
10.30.0.0/16
│
├─ AZ-a (ap-northeast-2a)
│   ├ public subnet
│   │   10.30.1.0/24
│   │
│   └ private subnet
│       10.30.11.0/24
│
├─ AZ-b (ap-northeast-2b)
│   ├ public subnet
│   │   10.30.2.0/24
│   │
│   └ private subnet
│       10.30.12.0/24
│
├ Internet Gateway
│
└ NAT Gateway <- 지금 단계에서는 만들지 않음
   (public subnet에 1개)
```
### 설정값

+ 리소스: VPC 등
+ 이름: 0305-net-01
+ IPv4 CIDR 블록: 10.30.0.0/16
+ IPv6 CIDR 블록: 없음
+ 테넌시: 기본값
+ 가용 영역 수 : 2
  + 퍼블릭 서브넷 1 : 10.30.10.0/24 (public-a)
  + 퍼블릭 서브넷 2 : 10.30.20.0/24 (public-b)
  + 프라이빗 서브넷 1 : 10.30.30.0/24 (private-a)
  + 프라이빗 서브넷 2 : 10.30.40.0/24 (private-b)
+ NAT 게이트웨이 : 없음 (과금방지 & 시간 소요로 Auto Scailing 전에 생성) or 리전별 - 신규 (= 1개)
+ VPC 엔드포인트 : S3 (Bastion을 Private subnet에 두지 않을 것이라 불필요하지만 실무에서 많이 사용)

```
Private EC2
   │
   ├─ S3 → VPC Endpoint (무료)
   │
   └─ Internet → NAT Gateway (유료)
```
+ DNS 옵션 : 모두 활성화 (안 켜면 나중에 apt update 실패 할 수 있음)
<img width="1275" height="518" alt="image" src="https://github.com/user-attachments/assets/c2259f56-e604-4799-8202-10d29bf0663d" />

<img width="1245" height="632" alt="image" src="https://github.com/user-attachments/assets/9e39b40d-56b0-45b5-924a-e50ac0581300" />

<img width="395" height="487" alt="image" src="https://github.com/user-attachments/assets/e494c5c2-86d7-4e41-9c58-9346e0c5d960" />

<img width="1320" height="795" alt="image" src="https://github.com/user-attachments/assets/79f1715c-5a66-49a6-8323-1debd2dee802" />


## 1-2. vpc 생성 점검

1) 현재 구조 (NAT 아직 안 만듦)    

```
VPC
0305-net-01
10.30.0.0/16
│
├ ap-northeast-2a
│  ├ public subnet
│  │ 10.30.10.0/24
│  │
│  └ private subnet
│     10.30.30.0/24
│
├ ap-northeast-2b
│  ├ public subnet
│  │ 10.30.20.0/24
│  │
│  └ private subnet
│     10.30.40.0/24
│
├ Internet Gateway
│
└ S3 VPC Endpoint
```

2) 자동으로 만들어진 것  

+ 라우팅 테이블
 0305-net-01-rtb-public   
 0305-net-01-rtb-private-2a   
 0305-net-01-rtb-private-2b   

+ 네트워크 연결
  0305-net-01-igw   
  0305-net-01-vpce-s3
---

## 2. 퍼블릭 서브넷 설정 : 퍼블릭 IPv4 주소 자동할당을 활성화하지 않으면 EC2 퍼블릭 IP없어서  ALB 테스트, SSH 접속이 불가능     

**VPC > 서브넷 > 서브넷 선택 > 작업 > 서브넷 설정 편집**

1) 0305-net-01-subnet-public1-ap-northeast-2a : 퍼블릭 IPv4 주소 할당 활성화

+ 자동 할당 IP 설정 : 퍼블릭 IPv4 주소 자동 할당 활성화 : 체크표시 켜기 -> 저장

<img width="1461" height="509" alt="image" src="https://github.com/user-attachments/assets/6c796b3e-96a7-4a48-9bbe-e049f5e47b1b" />

<img width="1274" height="421" alt="image" src="https://github.com/user-attachments/assets/bc3a3c07-9ce6-4ed7-9433-2e89584522e9" />

<img width="655" height="283" alt="image" src="https://github.com/user-attachments/assets/004ba2c6-5bc8-4d5d-a041-b3a31758451d" />

<img width="1265" height="463" alt="image" src="https://github.com/user-attachments/assets/fb5807f7-85f9-4678-87c0-f94c139279cd" />


2) 0305-net-01-subnet-public2-ap-northeast-2b : 퍼블릭 IPv4 주소 할당 활성화

+ public1과 같은 방식으로 활성화 -> 저장

<img width="932" height="222" alt="image" src="https://github.com/user-attachments/assets/e75e6698-f399-4b35-a093-7448cfe3cd07" />

<img width="1262" height="460" alt="image" src="https://github.com/user-attachments/assets/a9842378-2d07-43a2-a8cd-22d9ee9eb106" />

## 3. 보안그룹


