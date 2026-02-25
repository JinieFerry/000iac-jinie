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
각 디렉토리	역할
+ vpc	          : VPC + Subnet만 생성
+ route-public	: IGW + Public RT
+ route-private	: Private RT

1-2. 서울 새구조 성공
<img width="1309" height="747" alt="image" src="https://github.com/user-attachments/assets/994066ae-6e8f-4529-b8b5-a2ed742555b2" />

```
  330  history
  331  # 실습2 각기 다른 디렉토리로 퍼블릭 인터넷용 1 + 내부용 3
  332  # 서울 디렉토리 생성
  333  cd /d/tg/vpcex/live/seoul
  334  mkdir vpc
  335  mkdir route-public
  336  mkdir route-private
  337  # vpc 디렉토리 기존 terragrunt.hcl 이동
  338  mv terragrunt.hcl vpc/
  339  # route-puublic 모듈 분리
  340  # modules/route-public/main.tf 생성
  341  ls
  342  cd ..
  343  ls
  344  cd ..
  345  ls
  346  cd modules/
  347  ls
  348  ls
  349  pwd
  350  ls
  351  mkdir route-public
  352  cd route-public/
  353  touch main.tf variable.tf
  354  mv variable.tf variables.tf
  355  ls
  356  # variables.tf 작성
  357  vi variables.tf
```
+ route-public/variables.tf


```
variable "vpc_id" {}
variable "subnet_id" {}
variable "region" {}
variable "name" {}
```
```
  359  # main.tf 작성
  360  vi main.tf
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

  361  cd ..
  362  ls
  # route-private 몯퓰 생성
  363  mkdir route-private
  364  cd route-private
  365  ls
  366  touch main.tf variables.tf
  367  ls
  368  # variables.tf 작성
  369  vi variables.tf
```
  + route-private/variables.tf

```
variable "vpc_id" {}
variable "subnet_ids" {}
variable "region" {}
variable "name" {}
```
  370  # main.tf 작성
  371  vi main.tf
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
  372  # vpc 모듈에서 output 빼기
  373  # modules.vpc로 이동
  374  cd /d/tg/vpcex/modules/vpc
  375  # outputs.tf 생성
  376  touch outputs.tf
  377  vi outputs.tf
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
  378  # subnet_10 : 퍼블릭 , subnet_20/30/40 : 프라이빗
  379  # live/seoul 구조 분리
  380  cd /d/tg/vpcex/live
  381  ls
  382  cd seoul/
  383  # 서울 구조 확인
  384  pwd
  385  ls -R
  386  # route-public 설정
  387  cd route-public/
  388  ls
  389  # terragrunt.hcl 작성
  390  touch terragrunt.hcl
  391  vi terrgrunt.hcl
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
  392  # route-private 설정
  393  cd ../route-private
  394  # terragrunt.hcl 작성
  395  touch terragrunt.hcl
  398  vi terragrunt.hcl
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
  402  cd route-public/
  407  # vpc/terragrunt.hcl에서 output 실제로 나오고 있는지 확인
  408  cd ..
  409  ls
  410  cd ..
  411  ls
  412  cd ..
  413  ls
  414  cd modules/
  415  ls
  416  cd vpc/
  417  ls
  418  vi outputs.tf
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
  478  # live/seoul 루트에 예전 .terragrunt-cache나 terraform.tfstate 남아있는지 확인
  479  cd /d/tg/vpcex/live/seoul
  480  ls -a
  481  user@DESKTOP-5AFO9PS MINGW64 /d/tg/vpcex/live/seoul
  482  $ # live/seoul 루트에 예전 .terragrunt-cache나 terraform.tfstate 남아있는지 확인
  483  ]
  484  user@DESKTOP-5AFO9PS MINGW64 /d/tg/vpcex/live/seoul
  485  $ cd /d/tg/vpcex/live/seoul
  486  user@DESKTOP-5AFO9PS MINGW64 /d/tg/vpcex/live/seoul
  487  $ ls -a
  488  ./  ../  .terraform.lock.hcl  .terragrunt-cache/  route-private/  route-public/  vpc/
  489  aws ec2 describe-vpcs --region ap-northeast-2 --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"
  490  aws ec2 describe-subnets   --region ap-northeast-2   --filters "Name=vpc-id,Values=vpc-0a2b1b262ef421122"   --query "Subnets[*].SubnetId"
  491  # seoul VPC ID 확인
  492  aws ec2 describe-vpcs --region ap-northeast-2 --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"
  493  # seoukl Subnet 확인
  494  aws ec2 describe-subnets   --region ap-northeast-2   --filters "Name=vpc-id,Values=vpc-0a2b1b262ef421122"   --query "Subnets[*].SubnetId"
  495  # 서울 기존 서브넷 하나씩 삭제
  496  aws ec2 delete-subnet --subnet-id subnet-0d58285edea403c10 --region ap-northeast-2
  497  aws ec2 delete-subnet --subnet-id subnet-05cfa652b9781cdfd --region ap-northeast-2
  498  aws ec2 delete-subnet --subnet-id subnet-0e9c65dde29930cec --region ap-northeast-2
  499  aws ec2 delete-subnet --subnet-id subnet-0e7d05ef75fe57e17 --region ap-northeast-2
  500  # vpc에 붙어있는 인터넷 게이트웨이 있는지 확인
  501  aws ec2 describe-internet-gateways   --region ap-northeast-2   --filters "Name=attachment.vpc-id,Values=vpc-0a2b1b262ef421122"   --query "InternetGateways[*].InternetGatewayId"
  502  # [] 안이 비어있으면 IGW 없는 것
  503  # 서올vpc에는 아직 lGW가 붙어있지 않으므로 VPC 삭제
  504  aws ec2 delete-vpc   --vpc-id vpc-0a2b1b262ef421122   --region ap-northeast-2
  505  # vpc 삭제 확인
  506  aws ec2 describe-vpcs   --region ap-northeast-2   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}"
  507  # 서울 새구조로 vpc 생성
  508  cd /d/tg/vpcex/live/seoul/vpc
  509  ls
  510  terragrunt apply
  511  pwd
  512  # public route 생성
  513  cd ../route-public
  514  terragrunt apply
  515  # private route생성
  516  cd ../route-private
  517  terragrunt apply
  518  # VPC 확인
  519  aws ec2 describe-vpcs   --region ap-northeast-2   --filters "Name=cidr,Values=10.200.0.0/16"   --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock}"
  520  # IGW 붙었는지 확인
  521  aws ec2 describe-internet-gateways   --region ap-northeast-2   --filters "Name=attachment.vpc-id,Values=vpc-0ed0901557c9b37b8"   --query "InternetGateways[*].InternetGatewayId"
  522  # 라우팅 테이블 확인
  523  aws ec2 describe-route-tables   --region ap-northeast-2   --filters "Name=vpc-id,Values=vpc-0ed0901557c9b37b8"   --query "RouteTables[*].{ID:RouteTableId,Routes:Routes[*].DestinationCidrBlock}"
  524  # public subnet이 IGW 타는지 확인
  525  aws ec2 describe-route-tables   --region ap-northeast-2   --filters "Name=association.subnet-id,Values=subnet-0b73b6fa2bd65abcd"
```
