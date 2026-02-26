# 2026.02.25.wed
# vpc 생성 실습 1 : 동일한 라우팅 테이블
## 1-1. 서울 성공
<img width="1474" height="814" alt="image" src="https://github.com/user-attachments/assets/d7465d54-229f-4e1f-b9aa-6db10e723c1f" />


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

## 2-1. 버지니아 성공
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
## 3-1. 유럽 : 파리 성공
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
  source = "../../../modules/vpc"
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

# VPC 생성 실습 2 : 인터넷(퍼블릭용) 1 + 3개의 내부용을 각가 다른 디렉토리로 아이피 테이블 잡기 (디스트로이를 개별적으로 할 수 있도록)

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
각 디렉토리	역할
+ vpc	          : VPC + Subnet만 생성
+ route-public	: IGW + Public RT
+ route-private	: Private RT

## 1-2. 서울 새구조 성공
<img width="1309" height="747" alt="image" src="https://github.com/user-attachments/assets/994066ae-6e8f-4529-b8b5-a2ed742555b2" />

```

# 실습2 각기 다른 디렉토리로 퍼블릭 인터넷용 1 + 내부용 3
# 서울 디렉토리 생성
cd /d/tg/vpcex/live/seoul
mkdir vpc
mkdir route-public
mkdir route-private

# vpc 디렉토리 기존 terragrunt.hcl 이동
mv terragrunt.hcl vpc/

# route-puublic 모듈 분리
# modules/route-public/main.tf 생성
# 해당 디랙토리로 이동
ls
cd ..
ls
cd ..
ls
cd modules/
pwd
ls

# route-public 디렉토리 생성
mkdir route-public

# main.ft , variable.tf 작성
cd route-public/
touch main.tf variable.tf
mv variable.tf variables.tf
ls

# variables.tf 작성
vi variables.tf
```

+ route-public/variables.tf

```
variable "vpc_id" {}
variable "subnet_id" {}
variable "region" {}
variable "name" {}
```

```
# main.tf 작성
vi main.tf
```

+ route-public/main.tf

```
provider "aws" {
  region = var.region
}

resource "aws_internet_gateway" "igw" {
  vpc_id = var.vpc_id
}

resource "aws_route_table" "public_rt" {
  vpc_id = var.vpc_id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "${var.name}-public-rt"
  }
}

resource "aws_route_table_association" "public_assoc" {
  subnet_id      = var.subnet_id
  route_table_id = aws_route_table.public_rt.id
}
```
```
# route-private 몯퓰 생성
cd ..
ls

mkdir route-private
cd route-private
ls

touch main.tf variables.tf
ls

# variables.tf 작성
vi variables.tf
```
 + route-private/variables.tf
```
variable "vpc_id" {}
variable "subnet_ids" {}
variable "region" {}
variable "name" {}
```
```
# main.tf 작성
vi main.tf
```
+ route-private/main.tf
```
provider "aws" {
  region = var.region
}

resource "aws_route_table" "private_rt" {
  vpc_id = var.vpc_id

  tags = {
    Name = "${var.name}-private-rt"
  }
}

resource "aws_route_table_association" "private_assoc" {
  for_each = toset(var.subnet_ids)

  subnet_id      = each.value
  route_table_id = aws_route_table.private_rt.id
}
```
```
# vpc 모듈에서 output 빼기
# modules.vpc로 이동
cd /d/tg/vpcex/modules/vpc

# outputs.tf 생성
touch outputs.tf
vi outputs.tf
```
  + output.tf
```
output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_id" {
  value = aws_subnet.subnet_10.id
}

output "private_subnet_ids" {
  value = [
    aws_subnet.subnet_20.id,
    aws_subnet.subnet_30.id,
    aws_subnet.subnet_40.id
  ]
}
```
```
# subnet_10 : 퍼블릭 , subnet_20/30/40 : 프라이빗
# live/seoul 구조 분리
cd /d/tg/vpcex/live
 ls

# 서울 구조 확인
cd seoul/
pwd
ls -R # tree 구조로 확인

# route-public 설정
cd route-public/
ls

# terragrunt.hcl 작성
touch terragrunt.hcl
vi terrgrunt.hcl
```

  + vpc/terragrunt.hcl
```
terraform {
  source = "../../../modules/vpc"
}

inputs = {
  region = "ap-northeast-2"
  cidr_block = "10.200.0.0/16"
  name = "krs"
}
```
```
# route-private 설정
 cd ../route-private
# terragrunt.hcl 작성
touch terragrunt.hcl
vi terragrunt.hcl
```
  + route-public/terragrunt.hcl
```
terraform {
  source = "../../../modules/route-public"
}

dependency "vpc" {
  config_path = "../vpc"
}

inputs = {
  region     = "ap-northeast-2"
  vpc_id     = dependency.vpc.outputs.vpc_id
  subnet_id  = dependency.vpc.outputs.public_subnet_id
  name       = "krs"
}
```
```
cd route-public/
# vpc/terragrunt.hcl에서 output 실제로 나오고 있는지 확인
cd ..
ls
cd ..
ls
cd ..
ls
cd modules/
ls
cd vpc/
ls
vi outputs.tf
```
  + /d/tg/vpcex/modules/vpc/output.tf
```
output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_id" {
  value = aws_subnet.subnet_10.id
}

output "private_subnet_ids" {
  value = [
    aws_subnet.subnet_20.id,
    aws_subnet.subnet_30.id,
    aws_subnet.subnet_40.id
  ]
}
```
```
# live/seoul 루트에 예전 .terragrunt-cache나 terraform.tfstate 남아있는지 확인
cd /d/tg/vpcex/live/seoul
ls -a
user@DESKTOP-5AFO9PS MINGW64 /d/tg/vpcex/live/seoul

# live/seoul 루트에 예전 .terragrunt-cache나 terraform.tfstate 남아있는지 확인
 cd /d/tg/vpcex/live/seoul
ls -a
```
```
# 안에 남아있는 것 확인되면 진행
./  ../  .terraform.lock.hcl  .terragrunt-cache/  route-private/  route-public/  vpc/

# seoul VPC ID 확인
aws ec2 describe-vpcs --region ap-northeast-2 --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"

# seoukl Subnet 확인
aws ec2 describe-subnets   --region ap-northeast-2   --filters "Name=vpc-id,Values=vpc-0a2b1b262ef421122"   --query "Subnets[*].SubnetId"

# 서울 기존 서브넷 하나씩 삭제
aws ec2 delete-subnet --subnet-id subnet-0d58285edea403c10 --region ap-northeast-2
aws ec2 delete-subnet --subnet-id subnet-05cfa652b9781cdfd --region ap-northeast-2
aws ec2 delete-subnet --subnet-id subnet-0e9c65dde29930cec --region ap-northeast-2
aws ec2 delete-subnet --subnet-id subnet-0e7d05ef75fe57e17 --region ap-northeast-2

# vpc에 붙어있는 인터넷 게이트웨이 있는지 확인
aws ec2 describe-internet-gateways   --region ap-northeast-2   --filters "Name=attachment.vpc-id,Values=vpc-0a2b1b262ef421122"   --query "InternetGateways[*].InternetGatewayId"

# [] 안이 비어있으면 IGW 없는 것
# 서올vpc에는 아직 lGW가 붙어있지 않으므로 VPC 삭제
aws ec2 delete-vpc   --vpc-id vpc-0a2b1b262ef421122   --region ap-northeast-2

# vpc 삭제 확인
aws ec2 describe-vpcs   --region ap-northeast-2   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"

# 서울 새구조로 vpc 생성
cd /d/tg/vpcex/live/seoul/vpc
ls
terragrunt apply
pwd

# public route 생성
cd ../route-public
terragrunt apply

# private route생성
cd ../route-private
terragrunt apply

# VPC 확인
aws ec2 describe-vpcs   --region ap-northeast-2   --filters "Name=cidr,Values=10.200.0.0/16"   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock}"

# IGW 붙었는지 확인
aws ec2 describe-internet-gateways   --region ap-northeast-2   --filters "Name=attachment.vpc-id,Values=vpc-0ed0901557c9b37b8"   --query "InternetGateways[*].InternetGatewayId"

# 라우팅 테이블 확인
aws ec2 describe-route-tables   --region ap-northeast-2   --filters "Name=vpc-id,Values=vpc-0ed0901557c9b37b8"   --query "RouteTables[*].{ID:RouteTableId,Routes:Routes[*].DestinationCidrBlock}"

# public subnet이 IGW 타는지 확인
aws ec2 describe-route-tables   --region ap-northeast-2   --filters "Name=association.subnet-id,Values=subnet-0b73b6fa2bd65abcd"
```

## 2-2. 버지니아 새 구조 성공 :

<img width="1311" height="747" alt="image" src="https://github.com/user-attachments/assets/b6a6ca61-8086-42c4-8987-051f0aed3f52" />

```
# 기존 버지니아 VPC 삭제
# 버지니아 vpc 조회
aws ec2 describe-vpcs   --region us-east-1   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"

# 버지니아 기존 서브넷 조회
aws ec2 describe-subnets   --region us-east-1   --filters "Name=vpc-id,Values=vpc-0ebe76aa930b4411c"   --query "Subnets[*].SubnetId"

# 버지니아 기존 서브넷 모두 삭제
aws ec2 delete-subnet --subnet-id subnet-016da3e0a3d99f660 --region us-east-1
aws ec2 delete-subnet --subnet-id subnet-0b90b9af3a9512302 --region us-east-1
aws ec2 delete-subnet --subnet-id subnet-0ea3b001491aa4e20 --region us-east-1
aws ec2 delete-subnet --subnet-id subnet-00d2f41850b094fbb --region us-east-1

 # 버지니아 IGW 확인
aws ec2 describe-internet-gateways   --region us-east-1   --filters "Name=attachment.vpc-id,Values=vpc-0ebe76aa930b4411c"   --query "InternetGateways[*].InternetGatewayId"

# IGW 없으므로 버지니아 기존 vpc 삭제
aws ec2 delete-vpc   --vpc-id vpc-0ebe76aa930b4411c   --region us-east-1

# 삭제 확인
aws ec2 describe-vpcs   --region us-east-1   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"

# 버지니아 새 구조 생성
pwd
mkdir vpc
mkdir route-public
mkdir route-private
ls

# 서울 그대로 가져와서 사용
cp /d/tg/vpcex/live/seoul/vpc/terragrunt.hcl vpc/
cp /d/tg/vpcex/live/seoul/route-public/terragrunt.hcl route-public/
cp /d/tg/vpcex/live/seoul/route-private/terragrunt.hcl route-private/

# 리전 수정
# 서울의 파일을 복사해왔으므로 리전 수정
# ap-northeast-2 (서울) => us-est-1 (버지니아)
# krs => krp

# vpc 리전 수정
cd /d/tg/vpcex/live/virginia/vpc
vi terragrunt.hcl
vi terragrunt.hcl

# 서울과 겹치지않도록 CIDR 수정
vi terragrunt.hcl

# vpc_cidr = "10.210.0.0/16" , subnet_base = "10.210"
# route-public 수정
cd ../route-public
vi terragrunt.hcl

# /ap-northeast-2  => "us-east-1" , /krs => /krp 로 수정
# route-private 수정
cd ../route-private
vi terragrunt.hcl

# 수정 확인
grep region terragrunt.hcl
grep name terragrunt.hcl
cd ..
ls
cd vpc/
ls
vi terragrunt.hcl

# 버지니아 새 구조vpc생성
terragrunt apply

# 퍼블릭 라우트
cd ../route-public
terragrunt apply

# 프라이빗 라우트
cd ../route-private
terragrunt apply
aws ec2 describe-vpcs   --region us-east-1   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"
```

## 3-2. 유럽 : 파리 새 구조 성공
<img width="1472" height="825" alt="image" src="https://github.com/user-attachments/assets/59ac9e9b-0c21-42c1-b425-1c01d0510a2b" />

```
 # 파리 새구조
# 파리 현재 상태 확인 : 기존 vpc 있는지
aws ec2 describe-vpcs   --region eu-central-1   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"

# 파리 디렉토리로 이동
cd /d/tg/vpcex/live/europe

# 디렉토리 생성
mkdir vpc
mkdir route-public
mkdir route-private

# 서울 설정 복사
user@DESKTOP-5AFO9PS MINGW64 /d/tg/vpcex/live/virginia/route-private

# 파리 새구조
user@DESKTOP-5AFO9PS MINGW64 /d/tg/vpcex/live/virginia/route-private

# 파리 현재 상태 확인 : 기존 vpc 있는지
user@DESKTOP-5AFO9PS MINGW64 /d/tg/vpcex/live/virginia/route-private
$ aws ec2 describe-vpcs   --region eu-west-3   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"
    [
        {         "ID": "vpc-0c1654f46ffd84056",;         "CIDR": "172.31.0.0/16",;         "Name": null;     }
    ]

# 서울 그대로 카피해서 사용
cp /d/tg/vpcex/live/seoul/vpc/terragrunt.hcl vpc/
cp /d/tg/vpcex/live/seoul/route-public/terragrunt.hcl route-public/
cp /d/tg/vpcex/live/seoul/route-private/terragrunt.hcl route-private/

# 서울의 설정 복사해왔으므로 리전 파리로 수정
ls
cd vpc/
ls
vi terragrunt.hcl
```
  + vpc/terragrunt.hcl
```
inputs = {
  vpc_name    = "krp-vpc"
  vpc_cidr    = "10.230.0.0/16"
  subnet_base = "10.230"
  region      = "eu-west-3"
}
```
```
cd ../route-public

# route-public 리전 수정
ls
vi terragrunt.hcl
```
+ route-public/terragrunt.hcl
```
region = "eu-west-3"
name   = "krp"
```
```
cd ../route-private
ls
vi terragrunt.hcl
```
+ route-private
```
region = "eu-west-3"
name   = "krp"
```
```
# route쪽 체크
cd /d/tg/vpcex/live/europe/route-public
grep region terragrunt.hcl
grep name terragrunt.hcl
cd ../route-private
grep region terragrunt.hcl
grep name terragrunt.hcl

# 파리는 vpc_name = "krp-vpc", vpc_cidr = "10.230.0.0/16" , subnet_base = ""10.230 , region = "eu-west-3"이면 정상

# 파리  apply
 # vpc
cd vpc
cd ..
ls
cd vpc/
terragrunt apply
cd ../route-public

# 퍼블릭 라우트
cd ../route-public
terragrunt apply

# 프라이빗 라우트
cd ../route-private
terragrunt apply
  
# 파리 확인
aws ec2 describe-route-tables   --region eu-west-3   --filters "Name=vpc-id,Values=vpc-066a8e960413307c4"   --query "RouteTables[*].{ID:RouteTableId,Routes:Routes[*].DestinationCidrBlock}"
```

  # 세 리전 모두 확인
```
cd ../live
cd ..
cd europe/
ls
aws ec2 describe-route-tables   --region <region>   --filters "Name=vpc-id,Values=<vpc-id>"   --query "RouteTables[*].{RT:RouteTableId,Routes:Routes[*].GatewayId}"

# 파리 리전 확인
aws ec2 describe-route-tables   --region us-east-1   --filters "Name=vpc-id,Values=vpc-05793db4815fe041c"   --query "RouteTables[*].{RT:RouteTableId,Routes:Routes[*].GatewayId}"
cd ..
ls


# 서울 리전 확인
cd seoul/

aws ec2 describe-route-tables   --region ap-northeast-2   --filters "Name=vpc-id,Values=vpc-0ed0901557c9b37b8"   --query "RouteTables[*].{RT:RouteTableId,Routes:Routes[*].GatewayId}"

# 버지니아 리전 확인
 cd ../virginia
aws ec2 describe-route-tables   --region us-east-1   --filters "Name=vpc-id,Values=vpc-05793db4815fe041c"   --query "RouteTables[*].{RT:RouteTableId,Routes:Routes[*].GatewayId}"
```

## 콘솔에서 보이는 세 리전의 라우팅 형식 달라보이지만 구조를 도식화하면 세 리전 모두 동일함
```
VPC (10.x.0.0/16)

Public Subnet
 └─ Public RT
     └─ 0.0.0.0/0 → IGW

Private Subnet 3개
 └─ Private RT

Main RT (기본, 실사용 거의 없음)
```
+ 🇰🇷 서울 (ap-northeast-2) 확인 로그 해석 : 정상
```
[
  { RT: rtb-02dce5ff5221e24ca, Routes: ["local"] },
  { RT: rtb-0aec17a056c6a851d, Routes: ["local"] },
  { RT: rtb-02582ba0490d91461, Routes: ["local", "igw-00826e9591b52ff9c"] }
]
```
rtb-02582ba0490d91461 → IGW 있음 →  Public RT

나머지 2개 → local만 있음 → Private RT + Main RT

+ 🇺🇸 버지니아 (us-east-1) 확인 로그 해석 : 정상
```
[
  { RT: rtb-029f4f8caa0eb2358, Routes: ["local", "igw-0d34af36de911bedb"] },
  { RT: rtb-087953c7843616d21, Routes: ["local"] },
  { RT: rtb-03e0e232c2222174c, Routes: ["local"] }
]
```
rtb-029f4f8caa0eb2358 → IGW 있음 →  Public RT

나머지 2개 → local만 →  Private + Main

+ 🇫🇷 파리 (eu-west-3) 확인 로그 해석 : 정상
```
[
  { RT: rtb-0fba960bed4d81633, Routes: ["local"] },
  { RT: rtb-0022e10c9c96cff41, Routes: ["local"] },
  { RT: rtb-02fcef7189dbb4532, Routes: ["local", "igw-0f02014b3168be42b"] }
]
```
rtb-02fcef7189dbb4532 → IGW 있음 →  Public RT

나머지 2개 → local만 →  Private + Main
  
+ 각 리전 마다 Route Table 3개, 그 중 한 개만 IGW 연결, 나머지는 2개는 내부용 (private + main)으로 구조 일치함

# 최종 실습 목적지 
```
                 Global Architecture

                Route53 (megaiac.com)
                        |
                ┌───────────────┐
                │   Asia Hub    │ (서울)
                │   TGW + NAT   │
                └──────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
     🇰🇷 Seoul       🇺🇸 US         🇪🇺 Europe
     Spoke EKS      Spoke EKS      Spoke EKS
```
+ 순서
Terragrunt Root → Network → EKS → IRSA → LBC → Workload

+ 방식
위 과정에서 만든 코드 기반으로 EKS까지 실습 (서울 스포크 완성 -> 외부 접속 성공)
```
Internet
   ↓
ALB (AWS Load Balancer)
   ↓
EKS Ingress
   ↓
Service
   ↓
Pod (Backend / Frontend)
```
  ## EKS 실습 시작
  ```
# 서울 네트워크 살아있는지 확인
aws ec2 describe-nat-gateways   --region ap-northeast-2
# NAT Gateway없으므로 서울 네트워크 다시 배포
## EKS 실습 시작

# 서울 vpc 다시 확정
cd awskr01/
ls

# /d/awskr01/terragrunt.hcl이 최상위 ROOT HCL 임
cd infrastructure/live/020-spokes/ap-northeast-2/network
ls

# terragrunt.hcl 보여야 정상
# 내용 확인
cat terragrunt.hcl

# EKS + LBC 가 동작하기 위해 서브넷에 있어야 하는 태그 포함시켜 파일 수정
cd /d/awskr01/infrastructure/modules/aws-network-spoke/main.tf
cd /d
cd /awskr01/infrastructure/modules/aws-network-spoke/main.tf
ls

cd awskr01/
ls
cd infrastructure/
ls
cd modules/
ls
cd aws-network-spoke/
ls
vi main.tf
```
+ aws-network-spoke/main.tf의 내용은 최종 아래와 같아야 함
```
########################################
# VPC
########################################

resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.name}-vpc"
  }
}

########################################
# Internet Gateway
########################################

resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id

  tags = {
    Name = "${var.name}-igw"
  }
}

########################################
# Public Subnets
########################################

resource "aws_subnet" "public" {
  count = length(var.public_subnets)

  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.public_subnets[count.index]
  availability_zone       = var.azs[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.name}-public-${count.index}"

    #  EKS 필수 태그
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/elb"                    = "1"
  }
}

########################################
# Private Subnets
########################################

resource "aws_subnet" "private" {
  count = length(var.private_subnets)

  vpc_id            = aws_vpc.this.id
  cidr_block        = var.private_subnets[count.index]
  availability_zone = var.azs[count.index]

  tags = {
    Name = "${var.name}-private-${count.index}"

    #  EKS 필수 태그
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"           = "1"
  }
}

########################################
# Public Route Table
########################################

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.this.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.this.id
  }

  tags = {
    Name = "${var.name}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```

## EKS + LBC 동작을 위한 서브넷 태그 추가
#EKS 클러스터 및 AWS Load Balancer Controller가 자동으로 서브넷을 인식하도록 하기 위해 퍼블릭 및 프라이빗 서브넷에 다음 태그를 추가함.

- kubernetes.io/cluster/megacluster = shared
- kubernetes.io/role/elb = 1 (Public Subnet)
- kubernetes.io/role/internal-elb = 1 (Private Subnet)

해당 태그 미존재 시 LoadBalancer 타입 서비스 생성 시 EXTERNAL-IP가 Pending 상태로 유지됨.

```
cd /d/awskr01/infrastructure/live/020-spokes/ap-northeast-2/network
terragrunt apply

# vpc apply
# 적용 후 확인
aws ec2 describe-subnets   --region ap-northeast-2   --query "Subnets[*].{ID:SubnetId,Tags:Tags}"
```
# 2026.02.25.수 수업 종료로 destroy
  ```
cd /d/awskr01/infrastructure/live/020-spokes/ap-northeast-2/network
 terragrunt destroy
aws ec2 describe-nat-gateways --region ap-northeast-2
history
  ```

### 내일 2026.02.26.thu 부터 05.500.0315 들어감

