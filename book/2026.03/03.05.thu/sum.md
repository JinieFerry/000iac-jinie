# 2026.03.05.thu AWS 콘솔 실습 로그

---

# 0. 실습 목표

이번 실습의 목표

- EC2 시작 템플릿 생성
- Nginx 자동 설치
- Instance ID 출력
- Auto Scaling Group 구성
- ALB 로드밸런싱 구성
- VPC 네트워크 구조 이해

---

# 1. 전체 아키텍처 구조
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
└─ NAT Gateway (이번 실습에서는 미사용)
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
Instance ID 출력
```

---

# 2. 네트워크 흐름

## 2.1 사용자 요청 흐름

```
User
│
▼
ALB
│
▼
Target Group
│
▼
EC2 (Auto Scaling)
│
▼
Nginx index.html
```



---

## 2.2 서버의 인터넷 접근
```
Private EC2
│
▼
NAT Gateway
│
▼
Internet
```

※ 이번 실습에서는 NAT Gateway는 생성하지 않았다.

---

# 3. 네트워크 레이어 구축

---

# 3.1 VPC 생성

### VPC 설정
```
VPC 이름
0305-net-01

CIDR
10.30.0.0/16
```

### 서브넷 구성
```
VPC
0305-net-01
10.30.0.0/16
│
├ ap-northeast-2a
│ ├ public subnet
│ │ 10.30.10.0/24
│ │
│ └ private subnet
│ 10.30.30.0/24
│
├ ap-northeast-2b
│ ├ public subnet
│ │ 10.30.20.0/24
│ │
│ └ private subnet
│ 10.30.40.0/24
│
├ Internet Gateway
│
└ S3 VPC Endpoint
```

---

## 생성 설정

- 리소스 : VPC 등
- 이름 : `0305-net-01`
- IPv4 CIDR : `10.30.0.0/16`
- IPv6 : 없음
- 가용영역 : 2개

### 서브넷
```
public-a 10.30.10.0/24
public-b 10.30.20.0/24

private-a 10.30.30.0/24
private-b 10.30.40.0/24
```


### NAT Gateway

이번 실습에서는 생성하지 않음    
(과금 방지)   



### VPC Endpoint     
S3 Endpoint 생성    

구조
```
Private EC2
│
├ S3 → VPC Endpoint
│
└ Internet → NAT Gateway (유료)
```

---

## DNS 옵션  

반드시 활성화   

```
enable DNS hostnames
enable DNS resolution
```

활성화하지 않으면   

```
apt update
yum update
```


실패할 수 있다.   

---  

# 3.2 퍼블릭 서브넷 설정   

퍼블릭 서브넷은 반드시    

Auto assign public IPv4    


활성화해야 한다.   

그렇지 않으면   

SSH 접속   
ALB 테스트    


모두 실패한다.   

---

설정 경로    

VPC    
→ Subnet    
→ Subnet 선택     
→ Edit subnet settings     

### 설정

Auto assign public IPv4     
Enabled    

public subnet 두 개 모두 설정

```
public-a     
public-b     
```


# 3.3 보안 그룹 구성

구조

```
Internet
│
▼
ALB (0305-sg-alb-01)
│
▼
EC2 (0305-sg-ec2-01)
```


외부 사용자는 EC2 직접 접근 불가, 오직 ALB → EC2만 허용된다.








