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

# 네트워크 단계
===

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
# 네트워크 레이어
---

## 1. VPC 생성
1) 0305-vpc-01 생성
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

## 2. 퍼블릭 서브넷 설정    
: 퍼블릭 IPv4 주소 자동할당을 활성화하지 않으면 EC2 퍼블릭 IP없어서  ALB 테스트, SSH 접속이 불가능     

```
                Internet
                    │
                    ▼
             Internet Gateway
                    │
        ┌───────────┴───────────┐
        │                       │
  Public Subnet A         Public Subnet B
   10.30.10.0/24           10.30.20.0/24
        │                       │
        │                       │
  Private Subnet A        Private Subnet B
   10.30.30.0/24           10.30.40.0/24
        │                       │
        └─────── S3 VPC Endpoint ───────┘
```

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

---

## 3. 보안그룹
**VPC > 보안그룹 > 보안그룹 생성**

Security Group은 같은 VPC 안에서만 서로 참조할 수 있다.

지금 우리가 만들 구조
```
Internet
   │
   ▼
ALB  (0305-sg-alb-01)
   │
   ▼
EC2  (0305-sg-ec2-01)
```
여기서 중요한 설정이
```
EC2 Security Group
source = 0305-sg-alb-01
```
인데 이게 같은 VPC에 있어야만 선택 가능하므로 보안그룹의 vpc는 0305-net-01-vpc로 통일한다.

1) ALB Security Group 생성

+ 기본
  + 이름 : 0305-sg-alb-01    
  + 설명 : 0305-sg-alb-01
  + VPC : 0305-net-01-vpc <- 보안그룹은 같은 vpc 안에서만 서로 참조 가능하므로 통일

+ 인바운드 규칙
  + 유형 :  HTTP
  + 포트 범위 : 80포트
  + 소스 : Anywhere-IPv4 0.0.0.0/0
 
+ 아웃바운드 규칙 : ALB는 뒤에 있느 EC2로 요청 보내야 함 = 기본값 그대로
  + 모든 트래픽
  + 0.0.0.0/0
    
<img width="711" height="277" alt="image" src="https://github.com/user-attachments/assets/fd156b6c-ceda-4c77-a714-040ed064e7e5" />

<img width="1303" height="210" alt="image" src="https://github.com/user-attachments/assets/9c368dd3-7816-46ee-b514-2fa40bb9d95b" />

<img width="1310" height="561" alt="image" src="https://github.com/user-attachments/assets/640ea477-cdc4-46f8-b1e7-dd6cc31b2595" />

<img width="1258" height="427" alt="image" src="https://github.com/user-attachments/assets/2fe9eae1-b157-4f36-983e-63edb418d260" />

2) EC2용 Security Group 생성
3) 
```
Internet
   │
   ▼
ALB (0305-sg-alb-01)
   │
   ▼
EC2 (0305-sg-ec2-01)
```

직접 접속 (Internet -> EC2)은 막는 구조      

+ 기본
  + 이름 : 0305-sg-ec2-01  
  + 설명 : 0305-sg-ec2-01
  + VPC : 0305-net-01-vpc <- 보안그룹은 같은 vpc 안에서만 서로 참조 가능하므로 통일

+ 인바운드 규칙 : ALB -> EC2만 허용
  + 유형 :  HTTP
  + 포트 범위 : 80포트
  + 소스 : 사용자 지정 -> 0305-sg-alb-01 (같은 vpc 안에 있어야 선택 가능)
 
+ 아웃바운드 규칙 : EC2 -> 외부 트래픽 허용. EC2는 여러 외부 리소스에 접근해야 할 수 있기 때문에 열어둠 (패키지 설치: yum / apt , S3 접근 , 외부 API 호출 )
  + 모든 트래픽
  + 프로토콜: 전체
  + 포트 범위: 전체
  + 대상 : 0.0.0.0/0

 <img width="708" height="272" alt="image" src="https://github.com/user-attachments/assets/8c835905-d518-40ca-9093-2a4209bc8bc9" />

<img width="1035" height="529" alt="image" src="https://github.com/user-attachments/assets/e2fef0e0-137f-4dd7-98a3-da827b6830a9" />

<img width="1308" height="841" alt="image" src="https://github.com/user-attachments/assets/9095e19d-a942-4eb1-b041-f6bea1ee86e9" />

<img width="1260" height="427" alt="image" src="https://github.com/user-attachments/assets/1c3acbef-6476-4711-8c45-266a6e818792" />


---
여기까지 네트워크 레이어 완료    

```
VPC
Subnets
IGW
S3 Endpoint
Route Tables
Security Groups
```
---
# 서비스 레이어
---
## 순서
```
Launch Template
→ Target Group
→ ALB
→ Auto Scaling
```

## 4. 시작 템플릿 Launch Template
**EC2 > 시작템플릿 > 시작 템플릿 생성**
+ 기본
   + 이름 : 0305-launch-template-01
   + 설명 : 0305-web-server
   + AMI : Amazon Linux 2023 kernel-6.1 AMI (EC2 타입을 보통 t3.micro , t3.small 쓰고 있엇 같은 x86 CPU라서 x86 AMI 써야 함 )
   + 인스턴스 유형 : t3.micro
   + 키 페어 : 없음 <- SSH 접속 안하고 웹서버 테스트만 하니까 필요 없음

```
실습 : Key Pair 없음
요즘 실무 : SSM 사용
옛날 방식 : SSH + Key Pair
```
+ 네트워크
   + 서브넷 : 시작 템플릿에 포함하지 않음
   + 가용 영역 : 시작 템플릿에 포함하지 않음
   + 방화벽 :기존 보안 그룹 선택 <- 아까 만든 0305-sg-ec2-01 써야 함

```
구조

Internet
   │
ALB (0305-sg-alb-01)
   │
EC2 (0305-sg-ec2-01)

보안 그룹 규칙
HTTP 80
source = 0305-sg-alb-01
```
즉, ALB만 EC2 접근 가능하다. 

+ 고급 세부 정보 > 사용자 데이터 : 아래 코드 붙여넣음

```
#!/bin/bash

dnf update -y
dnf install -y nginx

systemctl start nginx
systemctl enable nginx

INSTANCE_ID=$(curl http://169.254.169.254/latest/meta-data/instance-id)

echo "<h1>Hello! from $INSTANCE_ID</h1>" > /usr/share/nginx/html/index.html
```     

### 위 코드가 하는 일 : EC2가 생성되면 자동으로       
1. nginx 설치
2. 웹서버 실행
3. index.html 생성
4. 인스턴스 ID 표시

브라우저에서 Hell from i-xxxxxxxx로 나옴 : ALB 테스트 시 새로고침하면 

```
Hello from i-1
Hello from i-2
```
위와 같이 바뀌므로 로드밸런싱 확인 가능하다.
+ #!/bin/bash  없으면 스크립트 실행 안됨
+ base64 체크박스 : 체크하면 안됨


<img width="952" height="838" alt="image" src="https://github.com/user-attachments/assets/edd9d0b4-94ed-400c-8ad4-8b06991cf9e3" />

<img width="1480" height="649" alt="image" src="https://github.com/user-attachments/assets/9f7f954e-eb55-4652-9330-72fd1aea57b4" />

<img width="714" height="489" alt="image" src="https://github.com/user-attachments/assets/855ccbec-b3fa-474f-a0c0-8fdf86e3d0bb" />

<img width="541" height="382" alt="image" src="https://github.com/user-attachments/assets/960abc61-b2c2-4386-8dd1-058f083b4386" />

<img width="1254" height="442" alt="image" src="https://github.com/user-attachments/assets/91d209a1-02fe-49c3-8098-3ba1221b260d" />

## 5. 타켓 그룹
```
Internet
   ↓
ALB
   ↓
Target Group
   ↓
EC2
```
현재 구조는 위와 같기 때문에 ALB는 EC2를 직접 모른다. 따라서 중간 연결이 필요하다. (ALB → TargetGroup → EC2)

**EC2 > 로드 밸런싱 > 대상 그룹 > 대상 그룹 생성**
+  설정
  + 대상 유형 : 인스턴스
  + 대상 그룹 이름 : 0305-tg-web-01
  + 프로토콜 : HTTP
  + 포트 : 80
  + VPC : 0305-net-01-vpc

+ 상태 검사 : 그대로 두기
  + 프로토콜 : HTTp
  + 상태 검사 경로 : /

<img width="864" height="792" alt="image" src="https://github.com/user-attachments/assets/930397cb-7e94-44ac-8b24-0576a4c8fd4a" />


<img width="1081" height="669" alt="image" src="https://github.com/user-attachments/assets/e0d36441-c60e-45d1-b058-41535b1435a6" />

+ 대상 등록 : 수정 없이 '다음' 클릭

<img width="1454" height="781" alt="image" src="https://github.com/user-attachments/assets/e306c4ec-cede-4224-8aec-cc7c08ce06a1" />

+ 검토 및 생성 : '대상 그룹 생성' 클릭
+<img width="1309" height="647" alt="image" src="https://github.com/user-attachments/assets/cf80ee95-cfaa-4832-b8a3-6b0b8b83e909" />


<img width="1251" height="658" alt="image" src="https://github.com/user-attachments/assets/3ab62ce3-94f3-4ff0-a3ed-a72fd0763337" />

#### 아직 ALB , Auto Scailing, EC2 없기 때문에 대상은 0으로 뜸. 나중에 구조가 이렇게 되면 자동으로 들어감.
```
Internet
   ↓
ALB
   ↓
Target Group
   ↓
Auto Scaling
   ↓
EC2 생성 → Target Group 자동 등록
```

## 5. ALB 생성
**EC2 > 로드 밸런싱 > 로드 밸런서 생성**

+ Application Load Balancer 선택
<img width="811" height="707" alt="image" src="https://github.com/user-attachments/assets/b2fce369-72e3-43a5-9ea6-f26e861ac94c" />

+ 기본
   + 이름 : 0305-alb-web-01
   + 체계 : 인터넷 경계 = 외부접속 허용 (Internet → ALB)
   + IP 주소 유형 : IPv4

<img width="862" height="554" alt="image" src="https://github.com/user-attachments/assets/1a39954e-42ee-4bc0-a926-a081d54ecbd0" />

+ 네트워크 매핑
   + VPC : 0305-net-01-vpc 선택 10.30.0.0/16
   + IP 풀 : 아무것도 선택하지 않음 (ALB는 자동으로 퍼블릭 IP할당 되니까 필요없음, IPAM은 대기업 네트워크 관리에서 사용)
   + 가용 영역 및 서브넷 :  ap-northeast-2a 과 ap-northeast-2b . 둘 다 체크
   + 서브넷 선택 : public subnet 두 개 각각 선택      
     + ap-northeast-2a (apne2-az1) : 0305 ~ pubic1 ~ : 10.30.10.0/24
     + ap-northeast-2b (apne2-az2) : 0305 ~ public2 ~ : 10.30.20.0/24 

<img width="1001" height="477" alt="image" src="https://github.com/user-attachments/assets/ab295746-d896-401e-82e8-42506b918a68" />

+ 보안 그룹
   + 보안 그룹 : 0305-sg-alb-01 선택
<img width="906" height="239" alt="image" src="https://github.com/user-attachments/assets/91132310-0a51-42d6-930d-a81d5c720ba1" />
    
전체 보안 구조는 아래와 같다. 외부 -> EC2 직접 접근을 차단하고, 외부 -> ALB만 접근 가능하다. (보안이 좋아지고, 트래픽 분산과 오토 스케일링이 가능하다.) 
```
Internet
   │
[SG: ALB]
   │
Application Load Balancer
   │
[SG: EC2]
   │
EC2 Instance
```
사용자가 EC2에 직접 접속하지 않고, ALB에 접속하기 때문에 외부 트래픽을 받는 보안그룹은 ALB에 붙어야 한다.

ALB 보안 그룹의 역할은 인터넷 -> ALB 접속 허용이다. 즉 사용자가 http://ALB-DNS 로 접속한다.

```
Inbound
HTTP 80
0.0.0.0/0
```
EC2 보안 그룹의 역할은 ALB → EC2 만 허용으로, 외부 사용자는 EC2에 직접 접근하지 못 한다.

+ 리스너 및 라우팅

+ 리스너
  + 프로토콜 : HTTP
  + 포트 : 80
+ 기본 작업
  + 라우팅 액션 : 대상 그룹으로 전달
  + 대상 그룹 : 0305-tg-web-01

<img width="1072" height="397" alt="image" src="https://github.com/user-attachments/assets/2dfd84c7-79e6-45d6-b10c-4dc9e2d446cb" />

+ 아래로는 설정을 건들지 않고 로드 밸런서를 생성한다.
<img width="1254" height="717" alt="image" src="https://github.com/user-attachments/assets/4957f5f2-9288-44d9-adab-c8f0240fe20c" />


## 6. 대상 그룹에 넣을 EC2 생성
**EC2 > 인스턴스 > 인스턴스 시작**

### 첫번째 EC2
+ 이름 및 태그
   + 이름 : 0305-ec2-web-01
     
+ 애플리케이션 및 OS 이미지
   + AMI : Amazon Linux 2023
     
<img width="866" height="750" alt="image" src="https://github.com/user-attachments/assets/31aa3614-230e-4a59-9a35-8b9753d971e6" />

+ 인스턴스 유형 : t3.micro
+ 키 페어 : 키페어 없이 진행 <- ssh 접속 테스트는 AWS 콘솔로

<img width="791" height="319" alt="image" src="https://github.com/user-attachments/assets/95453cc4-f89e-42ac-b64b-eb7ce6d9a473" />

+ 네트워크 설정
   + VPC : 편집 눌러서 0305-net-01-vpc (10.30.0.0/16)으로 설정
   + 서브넷 (첫번째 EC2) : 0305-net-01-subnet-private1 (10.30.10.0/24)
   + 퍼블릭 IP 자동 할당 : 비활성화
   + 방화벽 : 기존 보안 그룹 선택
   + 일반 보안 그룹 : 0305-sg-ec2-01 

<img width="791" height="482" alt="image" src="https://github.com/user-attachments/assets/987d61b5-dd9d-4406-8bae-6960fe2a7f24" />

<img width="661" height="429" alt="image" src="https://github.com/user-attachments/assets/62cb9272-1007-4a6c-9df6-3b66619bbdc7" />

+ 고급 네트워크 구성, 스토리지 구성 건너뜀
+ 고급 세부 정보
   + 사용자 데이터 (첫번째 EC2): 웹서버 자동 설치용

```     
#!/bin/bash
dnf install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1> FERRY's WEB01</h1>" > /var/www/html/index.html
```
+ 인스턴스 시작 클릭
  
<img width="1303" height="470" alt="image" src="https://github.com/user-attachments/assets/4495ab98-0f14-40f1-990a-a17cc5906a1c" />

<img width="1300" height="807" alt="image" src="https://github.com/user-attachments/assets/7ed023db-78b5-4b58-9dc1-bc6f75f47b29" />

### 두번째 EC2
+ 이름 및 태그
   + 이름 : 0305-ec2-web-02
     
+ 애플리케이션 및 OS 이미지
   + AMI : Amazon Linux 2023
<img width="867" height="751" alt="image" src="https://github.com/user-attachments/assets/f04729f4-879b-48fe-a917-3d7d56b1480d" />

+ 인스턴스 유형 : t3.micro
+ 키 페어 : 키페어 없이 진행 <- ssh 접속 테스트는 AWS 콘솔로
<img width="865" height="319" alt="image" src="https://github.com/user-attachments/assets/91588674-f6f7-4ee2-8a5e-7d74649cc3b2" />

+ 네트워크 설정
   + VPC : 편집 눌러서 0305-net-01-vpc (10.30.0.0/16)으로 설정
   + 서브넷 (두번째 EC2) : 0305-net-01-subnet-private2 (10.40.10.0/24) <- AZ분산
   + 퍼블릭 IP 자동 할당 : 비활성화
   + 방화벽 : 기존 보안 그룹 선택
   + 일반 보안 그룹 : 0305-sg-ec2-01
<img width="719" height="483" alt="image" src="https://github.com/user-attachments/assets/6f426bc3-54c1-406e-8042-d919c6a74d08" />

<img width="701" height="483" alt="image" src="https://github.com/user-attachments/assets/0789dc23-ad1e-4b61-bb1f-2c3c8c8ad9c2" />

+ 고급 네트워크 구성, 스토리지 구성 건너뜀
+ 고급 세부 정보
   + 사용자 데이터 (두번째 EC2): 웹서버 자동 설치용
```
#!/bin/bash
dnf install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>FERRY'S WEB02</h1>" > /var/www/html/index.html
```
+ 인스턴스 시작 클릭
  
<img width="586" height="382" alt="image" src="https://github.com/user-attachments/assets/fa425834-bb48-4047-bd7a-ae3d84896f29" />

<img width="1301" height="489" alt="image" src="https://github.com/user-attachments/assets/921bd835-67f9-467d-91b7-020ee9b25b58" />

<img width="1299" height="805" alt="image" src="https://github.com/user-attachments/assets/b8f06c25-3a98-4db9-989d-5f032eb4b52e" />

## 7. 대상 그룹에 EC2 등록
현재 구조는 아래와 같다.   

```
Internet
   │
ALB (HTTP:80)
   │
Target Group
0305-tg-web-01
   │
EC2 Instances
```
그래서 ALB가 하는 일은 아래와 같다.    

```
http://ALB주소

요청 받음
   ↓
0305-tg-web-01으로 전달
   ↓
EC2 중 하나로 트래픽 분산
```
따라서 로드밸런서 생성 후에는 EC2를 반드시 Target Group에 등록해야 한다. 안그러면 ALB는 만들어져도 연결할 서버가 없는 상태가 된다.     

**EC2 > 대상그룹 > 0305-tg-web-01 선택 > 대상 등록 클릭**

<img width="1454" height="163" alt="image" src="https://github.com/user-attachments/assets/1166b95b-f442-4b18-a972-b29d3f15b6ab" />

<img width="1248" height="600" alt="image" src="https://github.com/user-attachments/assets/36feffd4-96ab-4aeb-9418-dd844f0cb47a" />



