# 2026.03.11.wed #

# 01. 작업 디렉토리 준비

```
# 현위치 확인
pwd
cd d
cd /d
ls

# 실습 폴더 생성
mkdir 0311-NatAlb
cd 0311-NatAlb/
pwd
```

# 1-2. root terragrunt.hcl 생성

```
#파일 생성
touch terragrunt.hcl

#파일 작성
vi terragrunt.hcl
```

```
locals {
  env_vars   = read_terragrunt_config(find_in_parent_folders("env.hcl"))
  aws_region = local.env_vars.locals.aws_region
}

generate "provider" {
  path      = "provider.tf"
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
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }

  config = {
    path = "${get_terragrunt_dir()}/terraform.tfstate"
  }
}
```

# 02. prod  디렉토리 생성

```
mkdir prod
cd prod
ls
```
# env.hcl 생성

```
vi env.hcl
```
### /d/0311-NatAlb/prod/env.hcl

```
locals {

environment = "prod"

aws_region = "ap-northeast-2"

}
```

# 04. VPC 디렉토리 생성

```
mkdir 01-hub-vpc
mkdir 02-service-vpc
mkdir 03-tgw
mkdir 04-alb
mkdir 05-ec2-nginx

#확인
ls
```

# 05. Hub VPC 생성

```
#이동
cd 01-hub-vpc/

vi terragrunt.hcl
```

###/d/0311-NatAlb/prod/01-hub-vpc/terragrunt.hcl
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

# 06. Service VPC 생성

```
cd ..
cd 02-service-vpc/
```

### /d/0311-NatAlb/prod/02-service-vpc/terragrunt.hcl
```
vi terragrunt.hcl
```

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

 ### /d/0311-NatAlb/prod/02-service-vpc/main.tf

```
vi main.tf
```

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

# 07. TGW 디렉토리
```
cd ..
cd 03-tgw/
```

### /d/0311-NatAlb/prod/03-tgw/terragrunt.hcl

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

# 08. 첫 실행

```
#항상 prod 루트에서 실행
cd ..
pwd
```

# 09. 초기화
`terragrunt run-all init`

# 10. 실행 계획
`terragrunt run-all plan`

# 11. 실제 생성

```
#중간 질문 yes 입력
terragrunt run-all apply
```

성공



# 코어 네트워크 구축 (Hub & Service VPC)

### /d/0311-NatAlb/prod/03-tgw/main.tf

```
pwd
touch main.tf
vi main.tf
```

```
variable "hub_vpc_id" { type = string }
variable "hub_subnet_ids" { type = list(string) }

variable "service_vpc_id" { type = string }
variable "service_subnet_ids" { type = list(string) }

resource "aws_ec2_transit_gateway" "main" {
  description = "Hub and Spoke TGW"

  tags = {
    Name = "main-tgw"
  }
}

resource "aws_ec2_transit_gateway_vpc_attachment" "hub" {
  subnet_ids         = var.hub_subnet_ids
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = var.hub_vpc_id

  tags = {
    Name = "hub-attachment"
  }
}

resource "aws_ec2_transit_gateway_vpc_attachment" "service" {
  subnet_ids         = var.service_subnet_ids
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = var.service_vpc_id

  tags = {
    Name = "service-attachment"
  }
}

output "tgw_id" {
  value = aws_ec2_transit_gateway.main.id
}

output "hub_attachment_id" {
  value = aws_ec2_transit_gateway_vpc_attachment.hub.id
}

output "service_attachment_id" {
  value = aws_ec2_transit_gateway_vpc_attachment.service.id
}
```
#03-tgw 단독으로 먼저 확인
```
terragrunt init
terragrunt plan
terragrunt apply
```

```
#  3개 생성 되어야 함
aws_ec2_transit_gateway.main

aws_ec2_transit_gateway_vpc_attachment.hub

aws_ec2_transit_gateway_vpc_attachment.service
```
<img width="1001" height="1042" alt="image" src="https://github.com/user-attachments/assets/2e1e1b5e-1625-4b13-8681-09f41c206b4a" />
<img width="1001" height="1042" alt="image" src="https://github.com/user-attachments/assets/72eda1c3-47f6-45db-af09-aceebd0b4542" />


## 삭제 전 콘솔에서 확인

+ vpc - transit gateway
<img width="997" height="1037" alt="image" src="https://github.com/user-attachments/assets/6a6d2538-3402-4e6f-b68e-c121d0f5e747" />
+ 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/7cda042a-42f5-44f9-892e-2ab15e1bacac" />

+ vpc - tgw attachments
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/a78942a0-71cd-492f-b783-4325c0fc38f9" />
+ 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/8cadac10-bef4-4a6f-8f20-c771db481fac" />

+ vpc
 + hub vpc
 + service vpc
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/9bec5ab0-5f48-4d18-af3c-f575d651e7fe" />
 + 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/acb12d7f-4950-4ce6-9669-13abadb27920" />

+ Nat Gateway
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/329b7a2b-29f8-4703-8336-ed7f8eb83cd8" />
+ 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/84b073b7-df18-4ef2-ba2a-a6cd17cc6c27" />

