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

2) 0305-net-01-subnet-public2-ap-northeast-2b : 퍼블릭 IPv4 주소 할당 활성화

+ public1과 같은 방식으로 활성화 -> 저장
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

2) EC2용 Security Group 생성
   
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

+ 대상 등록 : 수정 없이 '다음' 클릭


+ 검토 및 생성 : '대상 그룹 생성' 클릭


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

+ 기본
   + 이름 : 0305-alb-web-01
   + 체계 : 인터넷 경계 = 외부접속 허용 (Internet → ALB)
   + IP 주소 유형 : IPv4


+ 네트워크 매핑
   + VPC : 0305-net-01-vpc 선택 10.30.0.0/16
   + IP 풀 : 아무것도 선택하지 않음 (ALB는 자동으로 퍼블릭 IP할당 되니까 필요없음, IPAM은 대기업 네트워크 관리에서 사용)
   + 가용 영역 및 서브넷 :  ap-northeast-2a 과 ap-northeast-2b . 둘 다 체크
   + 서브넷 선택 : public subnet 두 개 각각 선택      
     + ap-northeast-2a (apne2-az1) : 0305 ~ pubic1 ~ : 10.30.10.0/24
     + ap-northeast-2b (apne2-az2) : 0305 ~ public2 ~ : 10.30.20.0/24 


+ 보안 그룹
   + 보안 그룹 : 0305-sg-alb-01 선택

    
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


+ 아래로는 설정을 건들지 않고 로드 밸런서를 생성한다.

## 6. 대상 그룹에 넣을 EC2 생성
**EC2 > 인스턴스 > 인스턴스 시작**

### 첫번째 EC2
+ 이름 및 태그
   + 이름 : 0305-ec2-web-01
     
+ 애플리케이션 및 OS 이미지
   + AMI : Amazon Linux 2023
+ 인스턴스 유형 : t3.micro
+ 키 페어 : 키페어 없이 진행 <- ssh 접속 테스트는 AWS 콘솔로
+ 네트워크 설정
   + VPC : 편집 눌러서 0305-net-01-vpc (10.30.0.0/16)으로 설정
   + 서브넷 (첫번째 EC2) : 0305-net-01-subnet-private1 (10.30.10.0/24)
   + 퍼블릭 IP 자동 할당 : 비활성화
   + 방화벽 : 기존 보안 그룹 선택
   + 일반 보안 그룹 : 0305-sg-ec2-01 
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
  
### 두번째 EC2
+ 이름 및 태그
   + 이름 : 0305-ec2-web-02
     
+ 애플리케이션 및 OS 이미지
   + AMI : Amazon Linux 2023
+ 인스턴스 유형 : t3.micro
+ 키 페어 : 키페어 없이 진행 <- ssh 접속 테스트는 AWS 콘솔로
+ 네트워크 설정
   + VPC : 편집 눌러서 0305-net-01-vpc (10.30.0.0/16)으로 설정
   + 서브넷 (두번째 EC2) : 0305-net-01-subnet-private2 (10.40.10.0/24) <- AZ분산
   + 퍼블릭 IP 자동 할당 : 비활성화
   + 방화벽 : 기존 보안 그룹 선택
   + 일반 보안 그룹 : 0305-sg-ec2-01
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

+ 정리

```
첫번째 EC2
10.30.30.19
private1 subnet
```
```
두번째 EC2
10.30.40.189
private2 subnet
```

```
ALB
 │
 ├─ EC2 #1 (10.30.30.x)
 └─ EC2 #2 (10.30.40.x)
```
위와 같은 구조로 AZ도 분산된 상태이다.

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

+ 대상 등록 > 사용 가능한 인스턴스
   + 0305-ec2-web-01과 0305-ec2-web-02 선택 
+ '아래에 보류 중인 것으로 포함' 클릭

+ '보류 중인 대상으로 등록' 클릭

## 8. ALB DNS 찾기
**EC2 > 로드밸런서 > 0305-alb-web-01 클릭**

+ DNS 이름 복사
```
0305-alb-web-01-531676862.ap-northeast-2.elb.amazonaws.com
```
## 9. 브라우저 접속 : 위에서 복사한 DNS 주소로 접속 

새로고침 여러번 하면 번갈아 나옴 : ALB 로드밸런싱 성공

```
FERRYS WEB01
FERRYS WEB02
FERRYS WEB01
FERRYS WEB02
```

## 10. Auto Scailing 그룹
**EC2 > Auto Scailing 그룹 > 생성**

+ 1단계 : 시작 템플릿 선택
   + 이름 : 0305-asg-web-01
   + 시작 템플릿 : 0305-launch-templete-01

+ 2단계 : 인스턴스 시작 옵션 선택
   + vpc : 0305-net-01-vpc
   + 가용영역 및 서브넷: ap-northeast-2a , 2b ~  public1,2만 선택 

+ 3단계 : 다른 서비스와 통합
  + 기존 로드 밸런서에 연결
  + 대상 그룹 선택 : 0305-th-web-01
  + 상태 확인 : Elastic Load Balancer 상태 확인 켜기만 체크

+ 4단계 : 그룹 크기 및 크기 조정 구성
   + 그룹 크기 : 원하는 용량 2
   + 크기 조정 : 최소 용량 2 , 최대 용량 4
```
평소 EC2 2개 유지
장애 나면 새로 생성
필요하면 최대 4개까지 확장
```
   + 인스턴스 유지 관리 정책 : 혼합동작 정책 없음
   + 추가 용량 설정 : 기본값
   + 추가설정 : 없음
          
+ 5단계 : 알림 추가 하지 않고 다음

+ 6단계 : 태그 추가 하지 않고 다음

## 11. Auto Scaling - Auto Healing 테스트

+ EC2 인스턴스 확인  
오토 스케일링 그룹을 생성하면 AWS가 EC2를 2개 자동생성한다. 

```
기존 EC2
0305-ec2-web-01
0305-ec2-web-02
        +
Auto Scaling이 새로 만든 EC2 2개
```

** 지금 있는 EC2 중 하나를 종료해보면 AWS 내부에서 현재 서버 수 = 1 , 최소 유지 수 = 2 라서 , 새 EC2를 자동 생성한다.**

+ EC2 하나 삭제

+ 자동으로 새 EC2 생성

## 12. ALB 접속 테스트
**EC2 > 로드밸런서 > 0305-alb-web-01 선택**
+ DNS 이름 복사 : 0305-alb-web-01-531676862.ap-northeast-2.elb.amazonaws.com
+ 브라우저 접속
  
Round Robin (새로고침 할 때 마다 바뀜)

현재 AWS 내부 구조는 다음과 같다.
```
사용자 (브라우저)
        ↓
ALB (Application Load Balancer)
        ↓
Target Group
        ↓
EC2 서버들
   WEB01
   WEB02
   AutoScaling 서버
```
그래서 브라우저가 요청 할 때 마다 ALB가 이렇게 처리한다.
```
요청1 → WEB02
요청2 → AutoScaling EC2
요청3 → WEB02 
요청4 → WEB01
요청5 → AutoScaling EC2
```
## 13. 실습 종료 후 리소스 삭제 (과금 방지)

AWS는 리소스를 삭제하지 않으면 계속 과금된다.     
특히 다음 리소스는 과금이 발생할 수 있다.       

EC2      
ALB     
NAT Gateway     
Elastic IP    

따라서 실습이 끝난 후에는 반드시 리소스를 삭제한다.    

삭제 순서는 의존성이 있기 때문에 아래 순서를 지켜야 한다.    

Auto Scaling    
→ EC2    
→ Load Balancer    
→ Target Group    
→ Launch Template   
→ VPC    

## 13-1. Auto Scaling 그룹 삭제

EC2 > Auto Scaling 그룹   

0305-asg-web-01    선택 → 삭제

삭제하면 Auto Scaling이 생성했던 EC2도 함께 종료된다. 
<i83-0ca23432d3cd" />

```
Auto Scaling EC2     
↓
자동으로 Terminated
```

## 13-2. EC2 인스턴스 삭제

EC2 > 인스턴스    

삭제 대상   

```
0305-ec2-web-01
0305-ec2-web-02
```
선택     

작업    
→ 인스턴스 상태   
→ 인스턴스 종료    

상태 확인     : Terminated  

+ 자동 생성 인스턴스 삭제

## 13-3. Load Balancer 삭제

EC2 > 로드 밸런서      

선택    

0305-alb-web-01  

삭제


ALB는 시간 단위로 과금되므로 반드시 삭제한다.

13-4. Target Group 삭제

EC2 > 대상 그룹

선택

0305-tg-web-01

삭제


13-5. Launch Template 삭제

EC2 > 시작 템플릿

선택

0305-launch-template-01

삭제


13-6. VPC 삭제 (선택)

네트워크까지 정리하려면 VPC도 삭제한다.

VPC > VPC

선택

0305-net-01-vpc

삭제


삭제되면서 같이 제거되는 리소스

Subnets
Route Tables
Security Groups
VPC Endpoint
Internet Gateway



13-7. 최종 과금 확인

마지막으로 다음 리소스가 0개인지 확인한다.

EC2
EC2 > 인스턴스
Running = 0


Load Balancer
EC2 > 로드 밸런서
0개



NAT Gateway
VPC > NAT Gateway
0개


실습 전체 구조 (정리)

이번 실습에서 만든 구조

Internet
   │
Internet Gateway
   │
Public Subnet
   │
ALB
   │
Target Group
   │
Auto Scaling Group
   │
EC2 (Nginx)

웹 요청 흐름

User
↓
ALB
↓
Target Group
↓
EC2
↓
index.html

로드밸런싱 확인

FERRYS WEB01
FERRYS WEB02
Hello from i-xxxx

새로고침할 때마다 서버가 바뀌면 정상이다.
