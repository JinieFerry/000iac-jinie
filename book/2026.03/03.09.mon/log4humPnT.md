사진과 텍스트가 있는 2026.03.09.mon 실습로그 : Git Bash & AWS console

Hub & Spoke Architecture
```
            Internet
               │
               ▼
           IGW
               │
               ▼
        Hub VPC (10.0.0.0/16)
        ┌───────────────┐
        │ ALB           │
        │ NAT Gateway   │
        └──────┬────────┘
               │
               ▼
         Transit Gateway
               │
               ▼
        Service VPC (10.1.0.0/16)
        ┌───────────────┐
        │ Private EC2   │
        │ Nginx Server  │
        └───────────────┘
```
# 2026.03.09.mon #
# 사전체크
```
terragrunt -version
terraform --version
aws --version
aws sts get-caller-identity
```

# 구조 확인
```
pwd
ls
tree -L 2
# 디렉토리로 이동
cd /d
ls
```
# 테라그런트 실습 디렉토리 : d 드라이브
# 테라그런트 실습 디렉토리로 이동
## 0.실습 프로젝트 폴더 생성
```
# 오늘자
mkdir 0306
ls
cd 0306/
ls
mkdir hubspoke
cd hubspoke/

# 환경 폴더 생성
mkdir prod
cd prod/
ls

# 실습 디렉토리 구조 생성
mkdir 01-hub-vpc
mkdir 02-service-vpc
mkdir 03-tgw
mkdir 03-2-routing
mkdir 04-alb
mkdir 05-ec2
```
## 0. env.hcl 생성
```
touch env.hcl
nano env.hcl
```
## env.hcl
```
locals {
  environment = "prod"
  aws_region  = "ap-northeast-2"
}
```
## 01. root terragrunt.hcl 생성 : 이 파일은 모든 하위 모듈 (01~05)이 공통으로 사용하는 설정이다.
```
#파일 위치는 /d/tg/terragrunt.hcl로 pod 폴더 밖이다.
#이동
cd /d/tg/0306
pwd

#파일생성
touch terragrunt.hcl
nano terragrunt.hcl
```
## 02. S3 Backend 생성 (콘솔) : Terraform 상태 파일 저장할 버킷이 필요하다. (AWS -> S3)
## 버킷 생성
<img width="878" height="554" alt="image" src="https://github.com/user-attachments/assets/834d1206-88c4-4f81-bd30-6ee5573da3ee" />

<img width="864" height="489" alt="image" src="https://github.com/user-attachments/assets/cc1ad388-9d5a-4f91-b45f-8db90d3a9872" />

<img width="870" height="822" alt="image" src="https://github.com/user-attachments/assets/c3906b6d-c265-4f3f-aef4-b39364e4d206" />

<img width="885" height="453" alt="image" src="https://github.com/user-attachments/assets/aeba727e-2b98-4226-9199-e4e0a740f5d4" />

## Dynamo DB Lock Table 생성 
: Terraform이 동시에 실행되면 state file 충돌이 생긴다. 그래서 terraform-locks 테이블로 state lock을 건다.     
**AWS 콘솔 → DynamoDB**

테이블 이름 : terraform-locks

설정

Partition key: LockID       Type: String
정렬키: 비워두기             Type: String    

그 외 옵션 건드리지 말고 생성    

<img width="893" height="583" alt="image" src="https://github.com/user-attachments/assets/5506cc60-9dc3-4063-acdb-3b809b721701" />

<img width="874" height="427" alt="image" src="https://github.com/user-attachments/assets/d7c70dd7-7081-4d3d-a5e1-935ab08f5d4d" />

<img width="863" height="735" alt="image" src="https://github.com/user-attachments/assets/36ba8f27-2f65-4ca7-845e-7fc1caa15004" />

### 다시 Git Bash로 이동
