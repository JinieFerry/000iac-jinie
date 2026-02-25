1. 서울 성공
<img width="1474" height="814" alt="image" src="https://github.com/user-attachments/assets/d7465d54-229f-4e1f-b9aa-6db10e723c1f" />

  # 2026.02.25.wed
  ## vpc 생성 실습 1 : 동일한 라우팅 테이블
  ```
  273  # 사전확인
  274  terraform --version
  275  aws --version
  276  aws sts get-caller-identity
  277  # 디랙토리 생성
  278  cd ~/tg
  279  cd~
  280  cd ~
  281  cd /tg
  282  cd ~/tg
  283  cd /d
  284  ls
  285  # 디렉토리 생성
  286  cd /d
  287  ls
  288  mkdir tg
  289  cd tg
  290  pwd
  291  mkdir -p vpcex/modules/vpc
  292  mkdir -p vpcex/live/seoul
  293  mkdir -p vpcex/live/virginia
  294  mkdir -p vpcex/live/europe
  295  tree
  296  ls
  297  # tree 대신 구조 확인
  298  ls -R
  299  find . -type d
  300  cp vpcex/modules/vpc
  301  cd vpcex/modules/vpc
  302  touch main.tf variables.tf
  303  vi variables.tf
```
+ variables.tf
```
variable "vpc_name" {}
variable "vpc_cidr" {}
variable "subnet_base" {}
variable "region" {}
```
+ main.tf
```
  304  vi main.tf
 ```
```
provider "aws" {
  region = var.region
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_subnet" "subnet_10" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "${var.subnet_base}.10.0/24"
}

resource "aws_subnet" "subnet_20" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "${var.subnet_base}.20.0/24"
}

resource "aws_subnet" "subnet_30" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "${var.subnet_base}.30.0/24"
}

resource "aws_subnet" "subnet_40" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "${var.subnet_base}.40.0/24"
}
```
```
  305  ls
  306  cd /d/tg/vpcex/live/seoul
  307  touch terragrunt.hcl
  308  vi terragrunt.hcl
```
+ terragrunt.hcl
```
  provider "aws" {
  region = var.region
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_subnet" "subnet_10" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "${var.subnet_base}.10.0/24"
}

resource "aws_subnet" "subnet_20" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "${var.subnet_base}.20.0/24"
}

resource "aws_subnet" "subnet_30" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "${var.subnet_base}.30.0/24"
}

resource "aws_subnet" "subnet_40" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "${var.subnet_base}.40.0/24"
}
```
```
  309  ls
  310  terragrunt init
  311  terragrunt apply
  312  history
```

2. 버지니아 성공
<img width="1471" height="823" alt="image" src="https://github.com/user-attachments/assets/5e6a35a8-211b-49df-b7b4-f3a167e99c92" />

```
  313  # 버지니아 시작
  315  # 설정 확인
  316  cd /d/tg/vpcex/live/virginia
  317  touch terragrunt.hcl
  318  vi terragrunt.hcl
```
+ terragrunt.hcl
```
terraform {
  source = "../../modules/vpc"
}

inputs = {
  vpc_name    = "krp-vpc"
  vpc_cidr    = "10.210.0.0/16"
  subnet_base = "10.210"
  region      = "us-east-1"
}
```
```
  319  # 버지니아 apply
  320  terragrunt init
  321  terragrunt apply
  322  history
```
3. 유럽 : 파리 성공
<img width="1469" height="829" alt="image" src="https://github.com/user-attachments/assets/2f445aa7-458a-49f0-ab35-919cd1f02a43" />

```
  323  # 유럽 시작
  324  cd /d/tg/vpcex/live/europe
  325  ls
  326  vi terragrunt.hcl
```
+ terragrunt.hcl

```
terraform {
  source = "../../modules/vpc"
}

inputs = {
  vpc_name    = "euw-vpc"
  vpc_cidr    = "10.220.0.0/16"
  subnet_base = "10.220"
  region      = "eu-west-3"
}
```
```
  327  # 유럽파리 apply
  328  terragrunt init
  329  terragrunt apply
  330  history
```

## VPC 생성 실습 2 : 인터넷(퍼블릭용) 1 + 3개의 내부용을 각가 다른 디렉토리로 아이피 테이블 잡기 (디스트로이를 개별적으로 할 수 있도록)

실습2를 위한 구조
```
/d/tg/vpcex
├── modules
│   └── vpc
│   └── route-public
│   └── route-private
│
└── live
    └── seoul
        ├── vpc
        │   └── terragrunt.hcl
        ├── route-public
        │   └── terragrunt.hcl
        └── route-private
            └── terragrunt.hcl
```
