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

  generate {
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }

  config {
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
### /d/0311-NatAlb/env.hcl

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



