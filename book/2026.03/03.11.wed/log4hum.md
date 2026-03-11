# 2026.03.11.wed  
# Terraform + Terragrunt Hub-Spoke 실습 정리 (NatAlb)

AWS **Hub & Spoke 중앙집중형 네트워크 아키텍처**를  
**Terragrunt + Terraform IaC 방식**으로 구축하는 실습 정리 문서.

본 문서는 아래 교안을 통합 정리한 **실습용 단일 문서**이다.

---

# 0. 아키텍처 개요


Internet
│
▼
ALB (Hub Public Subnet)
│
▼
Transit Gateway
│
▼
Service VPC (Private Subnet)
│
▼
EC2 (Nginx)


Outbound 흐름


EC2
→ TGW
→ Hub VPC
→ NAT Gateway
→ Internet


---

# 1. CIDR 설계 규칙

Hub VPC  


10.150.0.0/16


Service VPC  


10.151.0.0/16


Hub Subnet  


Public
10.150.21.0/24
10.150.23.0/24

Private
10.150.101.0/24
10.150.103.0/24


Service Subnet  


Private
10.151.101.0/24
10.151.103.0/24


주의 교안에 아래 CIDR이 남아있을 수 있음  


10.0.0.0/16
10.1.0.0/16


=> 반드시 10.150 / 10.151 로 수정  

---

# 2. 디렉터리 생성

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
│ ├ terragrunt.hcl
│ └ main.tf
│
├ 02-service-vpc
│ ├ terragrunt.hcl
│ └ main.tf
│
├ 03-tgw
│ ├ terragrunt.hcl
│ └ main.tf
│
├ 04-alb
│ ├ terragrunt.hcl
│ └ main.tf
│
└ 05-ec2-nginx
├ terragrunt.hcl
└ main.tf
```

---

# 3. Root Terragrunt 설정

파일  


NatAlb/terragrunt.hcl
```
locals {
env_vars = read_terragrunt_config(find_in_parent_folders("env.hcl"))
aws_region = local.env_vars.locals.aws_region
}

generate "provider" {
path = "provider.tf"
if_exists = "overwrite_terragrunt"

contents = <<EOF
provider "aws" {
region = "${local.aws_region}"
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
path = "${get_terragrunt_dir()}/terraform.tfstate"
}

}
```

---

# 4. 환경 변수 설정

파일  


prod/env.hcl
```

locals {

environment = "prod"

aws_region = "ap-northeast-2"

}

```
---

# 5. Hub VPC 구축

### terragrunt.hcl

```
include "root" {
path = find_in_parent_folders()
}

terraform {
source = "."
}

inputs = {

vpc_name = "hub-vpc"

vpc_cidr = "10.150.0.0/16"

}
```

### main.tf

```
variable "vpc_name" { type = string }
variable "vpc_cidr" { type = string }

module "hub_vpc" {

source = "terraform-aws-modules/vpc/aws"
version = "5.0.0"

name = var.vpc_name
cidr = var.vpc_cidr

azs = [
"ap-northeast-2a",
"ap-northeast-2c"
]

public_subnets = [
"10.150.21.0/24",
"10.150.23.0/24"
]

private_subnets = [
"10.150.101.0/24",
"10.150.103.0/24"
]

enable_nat_gateway = true
single_nat_gateway = true

}

output "vpc_id" { value = module.hub_vpc.vpc_id }
output "public_subnets" { value = module.hub_vpc.public_subnets }
output "private_subnets" { value = module.hub_vpc.private_subnets }
output "private_route_table_ids" { value = module.hub_vpc.private_route_table_ids }
output "public_route_table_ids" { value = module.hub_vpc.public_route_table_ids }
```

---

# 6. Service VPC 구축

### terragrunt.hcl

```
include "root" {
path = find_in_parent_folders()
}

terraform {
source = "."
}

inputs = {

vpc_name = "service-vpc"

vpc_cidr = "10.151.0.0/16"

}
```

### main.tf

```
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
"10.151.101.0/24",
"10.151.103.0/24"
]

enable_nat_gateway = false

}

output "vpc_id" { value = module.service_vpc.vpc_id }
output "private_subnets" { value = module.service_vpc.private_subnets }
output "private_route_table_ids" { value = module.service_vpc.private_route_table_ids }
```

---

# 7. Transit Gateway 구축

### terragrunt.hcl

```
include "root" { path = find_in_parent_folders() }

dependency "hub" {
config_path = "../01-hub-vpc"
}

dependency "service" {
config_path = "../02-service-vpc"
}

terraform { source = "." }

inputs = {

hub_vpc_id = dependency.hub.outputs.vpc_id
hub_subnet_ids = dependency.hub.outputs.private_subnets

service_vpc_id = dependency.service.outputs.vpc_id
service_subnet_ids = dependency.service.outputs.private_subnets

}
```

### main.tf

```
variable "hub_vpc_id" { type = string }
variable "hub_subnet_ids" { type = list(string) }

variable "service_vpc_id" { type = string }
variable "service_subnet_ids" { type = list(string) }

resource "aws_ec2_transit_gateway" "main" {

description = "Hub and Spoke TGW"

}

resource "aws_ec2_transit_gateway_vpc_attachment" "hub" {

subnet_ids = var.hub_subnet_ids
transit_gateway_id = aws_ec2_transit_gateway.main.id
vpc_id = var.hub_vpc_id

}

resource "aws_ec2_transit_gateway_vpc_attachment" "service" {

subnet_ids = var.service_subnet_ids
transit_gateway_id = aws_ec2_transit_gateway.main.id
vpc_id = var.service_vpc_id

}
```

---

# 8. ALB 구축

### terragrunt.hcl

```
include "root" { path = find_in_parent_folders() }

dependency "hub" {

config_path = "../01-hub-vpc"

}

terraform { source = "." }

inputs = {

vpc_id = dependency.hub.outputs.vpc_id
public_subnets = dependency.hub.outputs.public_subnets

}
```

### main.tf

```
variable "vpc_id" { type = string }
variable "public_subnets" { type = list(string) }

resource "aws_security_group" "alb_sg" {

name = "hub-alb-sg"

vpc_id = var.vpc_id

ingress {

from_port = 80
to_port = 80
protocol = "tcp"

cidr_blocks = ["0.0.0.0/0"]

}

egress {

from_port = 0
to_port = 0
protocol = "-1"

cidr_blocks = ["10.151.0.0/16"]

}

}
```

---

# 9. EC2 Nginx 구축

### terragrunt.hcl

```
include "root" { path = find_in_parent_folders() }

dependency "service" {

config_path = "../02-service-vpc"

}

terraform { source = "." }

inputs = {

vpc_id = dependency.service.outputs.vpc_id
subnet_id = dependency.service.outputs.private_subnets[0]

}
```

### main.tf

```
variable "vpc_id" { type = string }
variable "subnet_id" { type = string }

resource "aws_instance" "nginx" {

ami = "ami-0c9c942bd7bf113a2"

instance_type = "t3.micro"

subnet_id = var.subnet_id

user_data = <<EOF
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl start nginx
systemctl enable nginx
echo "Welcome to Centralized Network Nginx!" > /var/www/html/index.html
EOF

}
```

---

# 10. 배포

```
cd NatAlb/prod


#Init  


terragrunt run-all init


#Plan  


terragrunt run-all plan


#Apply  


terragrunt run-all apply

```
---

# 11. 테스트

브라우저 접속  


http://ALB_DNS


정상 결과  


Welcome to Centralized Network Nginx!


---

# 12. 리소스 삭제

과금 방지  


terragrunt run-all destroy


---

# END
Terraform Hub-Spoke 실습 클린 정리
