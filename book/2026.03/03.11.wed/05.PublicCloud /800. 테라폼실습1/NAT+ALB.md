# 중앙 집중형 네트워크 아키텍처 구축
### Terragrunt 기반 NatAlb 프로젝트 실습 교재

**IP 직업훈련 전문가 과정**

- IaC (Infrastructure as Code)
- AWS Enterprise Architecture
- Terragrunt + Terraform

---

# 1. 프로젝트 소개 및 아키텍처 개요

본 과정에서는 **Hub & Spoke 기반 네트워크 아키텍처**를  
**Terragrunt + Terraform IaC 방식**으로 구축한다.

프로젝트 루트 디렉터리

```
NatAlb
```

---

## 과금 주의

실습 아키텍처는 AWS 프리티어 범위를 초과한다.

### 주요 과금 리소스

| 서비스 | 과금 방식 |
|---|---|
| TGW | 시간 + 트래픽 |
| NAT Gateway | 시간 + 데이터 |
| ALB | LCU 기반 |
| EC2 | 시간 |

실습 종료 후 반드시 리소스 삭제

```
terragrunt run-all destroy
```

---

# 2. 디렉터리 구조

Terragrunt 환경 분리 + Terraform 모듈화를 결합한 Hybrid Module 구조

```
NatAlb/
│
├ terragrunt.hcl
│
└ prod/
   │
   ├ env.hcl
   │
   ├ 01-hub-vpc
   │   ├ terragrunt.hcl
   │   └ main.tf
   │
   ├ 02-service-vpc
   │
   ├ 03-tgw
   │
   ├ 04-alb
   │
   └ 05-ec2-nginx
```

### 의존성 흐름

```
01 hub vpc
        ↓
02 service vpc
        ↓
03 tgw
        ↓
04 alb
        ↓
05 ec2 nginx
```

각 디렉터리는 독립적인 terraform state를 가진다.

---

# 3. 글로벌 설정

## NatAlb/terragrunt.hcl

```hcl
locals {
  env_vars   = read_terragrunt_config(find_in_parent_folders("env.hcl"))
  aws_region = local.env_vars.locals.aws_region
}

generate "provider" {
  path = "provider.tf"

  if_exists = "overwrite_terragrunt"

  contents = <<EOF
provider "aws" {
  region = "$${local.aws_region}"
}
EOF
}

remote_state {

  backend = "local"

  generate = {
    path = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }

  config = {
    path = "$${get_terragrunt_dir()}/terraform.tfstate"
  }

}
```

---

## prod/env.hcl

```hcl
locals {

  environment = "prod"

  aws_region = "ap-northeast-2"

}
```

---

## DRY 구조

Terragrunt는

```
read_terragrunt_config
find_in_parent_folders
```

를 통해 상위 설정을 자동 상속한다.

---

# 4. Hub VPC 구축

Hub VPC는 다음 역할을 담당한다.

- NAT
- Internet Gateway
- 중앙 라우팅

---

## terragrunt.hcl

```
NatAlb/prod/01-hub-vpc/terragrunt.hcl
```

```hcl
include "root" {

  path = find_in_parent_folders()

}

terraform {

  source = "."

}

inputs = {

  vpc_name = "hub-vpc"

  vpc_cidr = "10.0.0.0/16"

}
```

---

## main.tf

```hcl
variable "vpc_name" { type = string }
variable "vpc_cidr" { type = string }

module "hub_vpc" {

  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = var.vpc_name

  cidr = var.vpc_cidr

  azs = [
    "ap-northeast-2a",
    "ap-northeast-2c"
  ]

  public_subnets = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]

  private_subnets = [
    "10.0.11.0/24",
    "10.0.12.0/24"
  ]

  enable_nat_gateway = true

  single_nat_gateway = true

}

output "vpc_id" {

  value = module.hub_vpc.vpc_id

}
```

---

# 5. Service VPC

서비스가 실행되는 격리된 VPC

```
EC2
EKS
RDS
```

---

## terragrunt.hcl

```
NatAlb/prod/02-service-vpc
```

```hcl
include "root" {

  path = find_in_parent_folders()

}

terraform {

  source = "."

}

inputs = {

  vpc_name = "service-vpc"

  vpc_cidr = "10.1.0.0/16"

}
```

---

## main.tf

```hcl
variable "vpc_name" { type = string }
variable "vpc_cidr" { type = string }

module "service_vpc" {

  source = "terraform-aws-modules/vpc/aws"

  version = "5.0.0"

  name = var.vpc_name

  cidr = var.vpc_cidr

  azs = [
    "ap-northeast-2a",
    "ap-northeast-2c"
  ]

  private_subnets = [
    "10.1.1.0/24",
    "10.1.2.0/24"
  ]

  enable_nat_gateway = false

}
```

---

# 6. Transit Gateway 구축

TGW는 Hub ↔ Service VPC 라우팅 허브 역할을 한다.

```
Service VPC
     │
     │
    TGW
     │
     │
Hub VPC
```

---

## TGW 생성

```hcl
resource "aws_ec2_transit_gateway" "main" {

  description = "Hub and Spoke TGW"

}
```

---

## Hub Attachment

```hcl
resource "aws_ec2_transit_gateway_vpc_attachment" "hub" {

  subnet_ids = var.hub_subnet_ids

  transit_gateway_id = aws_ec2_transit_gateway.main.id

  vpc_id = var.hub_vpc_id

}
```

---

## Service Attachment

```hcl
resource "aws_ec2_transit_gateway_vpc_attachment" "service" {

  subnet_ids = var.service_subnet_ids

  transit_gateway_id = aws_ec2_transit_gateway.main.id

  vpc_id = var.service_vpc_id

}
```

---

# 7. ALB 구축

ALB는 외부 트래픽 진입점

```
Internet
   ↓
ALB
   ↓
EC2
```

---

## ALB Security Group

```hcl
resource "aws_security_group" "alb_sg" {

  name = "hub-alb-sg"

  vpc_id = var.vpc_id

  ingress {

    from_port = 80

    to_port = 80

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]

  }

}
```

---

## ALB Module

```hcl
module "alb" {

  source = "terraform-aws-modules/alb/aws"

  version = "~> 9.0"

  name = "hub-central-alb"

  load_balancer_type = "application"

}
```

---

# 8. EC2 Nginx 배포

EC2는 Private Subnet에서 실행

SSH 대신 **SSM 접속 사용**

---

## IAM Role

```hcl
resource "aws_iam_role" "ssm_role" {

  name = "nginx-ssm-role"

}
```

---

## EC2

```hcl
resource "aws_instance" "nginx" {

  ami = "ami-0c9c942bd7bf113a2"

  instance_type = "t3.micro"

  subnet_id = var.subnet_id

}
```

---

# 9. 전체 배포

```
cd NatAlb/prod
```

---

## 초기화

```
terragrunt run-all init
```

---

## 계획 확인

```
terragrunt run-all plan
```

---

## 배포

```
terragrunt run-all apply
```

---

# 10. 배포 검증

출력된 ALB DNS 접속

```
http://ALB_DNS
```

정상 결과

```
Welcome to Centralized Network Nginx!
```

---

# 11. 리소스 삭제

실습 종료 후 반드시 실행

```
terragrunt run-all destroy
```

---

# 12. Cold Start 문제

Terragrunt 의존성 구조에서 처음 배포 시

```
detected no outputs
```

에러 발생 가능

---

## 해결 방법

dependency block에서

```
mock_outputs
```

사용

예시

```hcl
dependency "hub" {

  config_path = "../01-hub-vpc"

  mock_outputs = {

    vpc_id = "vpc-mock"

  }

}
```

---

# 13. Plan 파일 사용 금지

초기 배포에서

```
terragrunt run-all apply tfplan
```

사용하면 Mock 값이 실제 API로 전달된다.

---

## 올바른 방식

```
terragrunt run-all apply
```

---

# 완료

이 실습은 **Enterprise Hub-Spoke 아키텍처 기반 IaC 구축 실습**이다.
