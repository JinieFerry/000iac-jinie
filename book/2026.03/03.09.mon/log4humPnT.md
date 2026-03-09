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
#AWS 콘솔에서 버킷 , Dynamo DB 테이블 생성 후 GIt Bash로 컴백
## 3. HUB VPC 생성
```
ls
cd ..
ls
cd 0306/
ls
cd hubspoke/
ls
cd prod/
ls
pwd

#hub vpc 작성
cd 01-hub-vpc/

nano terragrunt.hcl
```
### terragrunt.hcl
```
include "root" {
  path = find_in_parent_folders()
}

terraform {
  source = "tfr:///terraform-aws-modules/vpc/aws?version=6.3.0"
}

inputs = {
  name = "hub-vpc"
  cidr = "10.0.0.0/16"

  azs = ["ap-northeast-2a", "ap-northeast-2c"]

  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.11.0/24", "10.0.12.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    Environment = "prod"
    Role        = "hub"
  }
}
```
```
#prod로 이동
cd ..

pwd

#첫 terragrunt 실행
terragrunt run-all init
terragrunt run-all plan
```

```
#prod로 이동
pwd
#실제 생성
#지금까지 한 것은 HUb VPC 생성 계획 확인단계로 아직 생성은 하지 않았음
```
## 04. 실제 생성
```
terragrunt run-all apply

```
### 콘솔에서 hub-vpc 생성 확인 : 성공
<img width="1717" height="190" alt="image" src="https://github.com/user-attachments/assets/d5e4ec9d-6210-4a6e-8f1b-79c2d08b08a5" />
자동 생성 되는 것:
```
public subnet
private subnet
NAT
IGW
route table
```
<img width="1722" height="371" alt="image" src="https://github.com/user-attachments/assets/35104b98-d7aa-4311-b1b8-a2c87fd58e64" />
<img width="1729" height="136" alt="image" src="https://github.com/user-attachments/assets/035b0072-bd5e-4436-b270-a1cb891b0872" />
<img width="1047" height="159" alt="image" src="https://github.com/user-attachments/assets/2c316a8c-4c88-4914-9361-28a2a8ffffbb" />
<img width="1721" height="275" alt="image" src="https://github.com/user-attachments/assets/095c0e96-edf9-4af9-846b-4b092dd873f7" />

*참고 (과금)

여기서 과금되는 건 NAT Gateway 하나다.    
서울 리전 시간당 약 $0.045 = 실습 끝나면 `terragrunt run-all destroy` 해야 한다.

현재 구조 : Hub 네트워크 중심 생성 완료 (env.hcl , terragrunt.hcl ✔ , S3 backend ✔ , DynamoDB lock ✔ , 01-hub-vpc ✔)
``` 
Internet
   │
   ▼
IGW
   │
   ▼
Hub VPC (10.0.0.0/16)
   ├ Public subnet
   ├ Public subnet
   ├ Private subnet
   └ Private subnet
```

---


## 05. Service VPC 생성
```
#위치
cd 02-service-vpc/
ls

# terragrunt.hcl 파일 작성
nano terragrunt.hcl
```
### 02-service-vpc/terragrunt.hcl
```
include "root" {
  path = find_in_parent_folders()
}

terraform {
  source = "tfr:///terraform-aws-modules/vpc/aws?version=6.3.0"
}

inputs = {
  name = "service-vpc"
  cidr = "10.1.0.0/16"

  azs = ["ap-northeast-2a", "ap-northeast-2c"]

  private_subnets = ["10.1.1.0/24", "10.1.2.0/24"]

  enable_nat_gateway = false

  tags = {
    Environment = "prod"
    Role        = "service"
  }
}
```
```
#prod로 이동
cd ..

#실행
terragrunt run-all plan

#실제 생성
terragrunt run-all apply
```

/d/tg/0309/hubspoke/prod
AWS 안에는 지금 이 구조
```
01-hub-vpc
02-service-vpc
03-tgw
03-2-routing
04-alb
05-ec2
env.hcl
```
Terraform 구조
```
hub-vpc      10.0.0.0/16
service-vpc  10.1.0.0/16
```
즉, 지금까지 : VPC 두 개 생성 완료
---

