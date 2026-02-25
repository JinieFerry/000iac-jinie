# log4hum-history.md
Project: awskr01
Author: JinieFerry
Environment: Windows 10 / Git Bash (MINGW64)
AWS Account: 219850556765 (IAM User: Ferry)
Region: ap-northeast-2

---

## Phase 0 – Tool Installation & Verification
```
choco install terraform -y
terraform --version
terragrunt --version
aws --version
```
---

## Phase 1 – GitHub Repository Setup
```
cd /d
mkdir awskr01
cd awskr01
git init
git branch -M main
git remote add origin https://github.com/JinieFerry/awskr01.git
git pull origin main
git remote -v
```
---

## Phase 2 – AWS CLI Configuration
```
aws configure
aws sts get-caller-identity
```
---

## Phase 3 – Infrastructure Directory Structure Creation
```
cd /d/awskr01

mkdir -p infrastructure/modules/prepare
mkdir -p infrastructure/modules/aws-network-spoke
mkdir -p infrastructure/live/000-prepare/ap-northeast-2
mkdir -p infrastructure/live/020-spokes/ap-northeast-2/network
```
---

## Phase 4 – Root Terragrunt Configuration

File:
infrastructure/live/terragrunt.hcl

```
remote_state {
  backend = "s3"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }
  config = {
    # 과금 주의: S3 버킷에 상태 파일이 저장되며, 저장된 용량 및 요청 횟수에 따라 소액의 종량제 요금이 발생합니다.
    bucket         = "awskr01-tfstate-apne2-12345" # 본인만의 고유한 이름으로 변경 필수
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "ap-northeast-2"
    encrypt        = true
    
    # 과금 주의: 동시성 제어를 위한 DynamoDB 테이블이 생성되며, 읽기/쓰기 용량에 따른 요금이 발생합니다.
    dynamodb_table = "awskr01-tflock-table"
  }
}

# 모든 리소스에 공통으로 들어갈 글로벌 태그 정의
inputs = {
  default_tags = {
    Project     = "awskr01-Hub-Spoke"
    ManagedBy   = "Terragrunt"
  }
}

```
Configured:
- S3 Backend
  bucket: awskr01-tfstate-apne2-jinie
  region: ap-northeast-2
- DynamoDB Lock Table
  table: awskr01-tflock-table
- default_tags defined

---

## Phase 5 – Prepare Module 작성

File:
infrastructure/modules/prepare/main.tf

```
# 1. ECR (Elastic Container Registry) 생성
# 프론트엔드와 백엔드의 도커(Docker) 이미지를 저장할 중앙 보관소입니다.
# 과금 주의: ECR은 저장된 이미지의 용량(GB당 월 $0.10) 및 데이터 전송량에 따라 과금이 발생합니다.

# 백엔드용 이미지 저장소 정의
resource "aws_ecr_repository" "backend_repo" {
  name                 = "awskr01-backend" # 저장소 이름 설정
  image_tag_mutability = "MUTABLE"           # 이미지 태그 변경 가능 여부 (MUTABLE: 동일 태그 덮어쓰기 허용)

  # 이미지 보안 설정
  image_scanning_configuration {
    scan_on_push = true # 이미지가 푸시(Push)될 때마다 보안 취약점을 자동으로 스캔함
  }
}

# 프론트엔드용 이미지 저장소 정의
resource "aws_ecr_repository" "frontend_repo" {
  name                 = "awskr01-frontend"
  image_tag_mutability = "MUTABLE"
}


# 2. EKS Cluster IAM Role
# 쿠버네티스 컨트롤 플레인(Control Plane)이 AWS 인프라(EC2, 로드밸런서 등)를 제어하기 위해 필요한 권한입니다.
# EKS 클러스터용 IAM Role(Identity and Access Management Role, 신뢰 관계 설정) 생성
resource "aws_iam_role" "eks_cluster_role" {
  name = "awskr01-eks-cluster-role"

# 이 역할(Role)을 누구(EKS 서비스)가 사용할 수 있는지 정의하는 신뢰 정책
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"   # sts 토큰,  역할을 맡을 수 있도록 허용하는 동작
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"  # EKS 서비스가 이 역할을 사용하도록 지정
      }
    }]
  })
}

# 생성한 역할에 실제 권한(Policy) 부여
# AWS IAM 역할에 특정 정책(Policy)을 부착(Attachment)하는 리소스 정의
resource "aws_iam_role_policy_attachment" "eks_cluster_policy" {
  # AmazonEKSClusterPolicy: EKS 클러스터가 AWS 리소스를 관리하는 데 필요한 표준 권한 ARN(Amazon Resource Name)
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  
  # 위에서 정의한 aws_iam_role.eks_cluster_role 리소스의 이름을 참조하여 해당 역할에 권한을 연결
  role       = aws_iam_role.eks_cluster_role.name 
}

```
Resources:
- aws_ecr_repository.backend_repo
- aws_ecr_repository.frontend_repo
- aws_iam_role.eks_cluster_role
- aws_iam_role_policy_attachment.eks_cluster_policy

---

## Phase 6 – Prepare Execution Layer 작성

File:
infrastructure/live/000-prepare/ap-northeast-2/terragrunt.hcl
```
# 현재 위치에서 상위 폴더를 거슬러 올라가며 최상위 terragrunt.hcl(S3/DynamoDB 설정)을 찾아 상속받음
include {
  path = find_in_parent_folders()
}

# 실제 인프라 리소스를 정의한 테라폼 모듈 소스 코드의 상대 경로를 지정
terraform {
  source = "../../../modules/prepare"
}

# 해당 모듈의 variables.tf에 정의된 변수들에 실제 값을 주입하는 블록
inputs = {
  # 리소스가 생성될 AWS 지역을 서울 리전으로 지정
  region      = "ap-northeast-2"
  
  # 현재 환경의 식별자 이름을 프로젝트 명칭에 맞춰 설정
  environment = "awskr01-prepare"
}
```
Execution:
```
cd infrastructure/live/000-prepare/ap-northeast-2
terragrunt init
terragrunt apply
```
Result:
ECR repositories created
IAM role created
Policy attached

---

## Phase 7 – Apply 중단 및 Lock 발생

terragrunt apply 실행 중 Ctrl + C

Result:
DynamoDB State Lock 발생

Error:
Error acquiring the state lock
ConditionalCheckFailedException

---

## Phase 8 – Lock 해제
```
terragrunt force-unlock 80e4f17d-aebc-39fe-57d1-f5ce1f38ca76
# yes
```
---

## Phase 9 – Prepare 재적용

terragrunt apply

Result:
Apply complete
Resources: 4 added

---

## Phase 10 – Network Module Skeleton 생성

Directory:
infrastructure/modules/aws-network-spoke/

Files:
variables.tf
main.tf
outputs.tf

---

## Phase 11 – Network Execution Layer 생성

File:
infrastructure/live/020-spokes/ap-northeast-2/network/terragrunt.hcl

terraform {
  source = "../../../../modules/aws-network-spoke"
}

---

## Phase 12 – Network Destroy 테스트
```
cd infrastructure/live/020-spokes/ap-northeast-2/network
terragrunt destroy -auto-approve
```
---

## Phase 13 – Prepare Destroy
```
cd infrastructure/live/000-prepare/ap-northeast-2
terragrunt destroy -auto-approve
```
---

## Phase 14 – Git Commit
```
cd /d/awskr01
git add .
git commit -m "feat: 온프레미스(10.10) 기준 글로벌 네트워크 설계 및 다중 가용영역 기반 하이브리드 인프라 구축 완료"
git push origin main
```
---

## End State

- Prepare resources destroyed
- Network skeleton ready
- Backend S3 + DynamoDB 유지
- GitHub main branch 최신 상태


user@DESKTOP-5AFO9PS MINGW64 /d/awskr01 (main)
 history

```
# 2026.02.24.tue #
## 05.130.0200.테라그란트 설치 ##
      choco --version
      choco install terraform -y
      terraform --version
      terragrunt --version

      brew tap hashicorp/tap
      brew tap hashicorp/tap
      
      brew tap hashicorp/tap

      Get-ExecutionPolicy
      cd ..
      ls
```
## 05.500.101.작업영역 만들기 ##

### D드라이브에 로컬 작업 디렉토리 생성
```
     cd /d
     mkdir awskr01
     cd awskr01
     pwd
     # Git 초기화
     git init
     git remote add origin https://github.com/JinieFerry/awskr01.git
     git branch -M main
     git pull origin main
```
### 프로젝트 최상위 경로(awskr01/)에 .gitignore 파일을 생성하고 아래 내용을 입력하여 저장
```
     ls
     vi .gitignore
```
+.gitingnore
```
# .gitignore 파일 내용
# 1. Terraform 및 Terragrunt 로컬 캐시 디렉토리
.terraform/
.terragrunt-cache/
**/.terraform/*
**/.terragrunt-cache/*

# 2. 인프라 실물 정보가 평문으로 담긴 상태 파일 (절대 업로드 금지)
*.tfstate
*.tfstate.*

# 3. 크래시 로그 및 재정의 파일
crash.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# 4. 자격 증명 및 로컬 환경 변수 파일
*.tfvars
*.tfvars.json
.env
secret.tfvars
credentials.csv

# 5. OS 생성 임시 파일 (macOS 및 Windows)
.DS_Store
Thumbs.db
```

### GitHub(깃허브) 원격 저장소(Remote Repository) 생성 및 연결
```
     git add .gitignore
     git commit -m "Update gitignore for Terraform security"
     git config --global user.name "JinieFerry"
     git config --global user.email "jinie.sej@gmail.com"
     git commit -m "Update gitignore for Terraform security"
```
### AWS CLI(Command Line Interface, 명령줄 인터페이스) 자격 증명 안전 구성 ###


     mv /c/Users/B1-MAIN/.pem/aws-bastion-key.pem /c/Users/user/.ssh/

     ls /c/Users/B1-MAIN/.pem

     ls /c/Users/user/Downloads

   61  ls /c/Users/user/.ssh
   62  cp /c/Users/B1-MAIN/.pem/aws-bastion-key.pem /c/Users/user/.ssh/
   63  cp /c/Users/B1-MAIN/.pem/aws-bastion-key.pem /c/Users/user/.ssh/
   64  chmod 400 /c/Users/user/.ssh/aws-bastion-key.pem
   65  ls /c/Users/user/.ssh
   66  chmod 400 /c/Users/user/.ssh/aws-bastion-key.pem
   67  ls -l /c/Users/user/.ssh
   68  chmod 400 /c/Users/user/.ssh/aws-bastion-key.pem
   69  ls -l /c/Users/user/.ssh
   70  git add .
   71  git commit -m "chore: 프로젝트 초기화 및 .gitignore 설정"
   72  git branch -M main
   73  git remote -v
   74  git push -u origin main
   75  aws configure
   76  aws --version
   77  aws --version
   78  aws --version
   79  aws configure
   80  aws sts get-caller-identity
   81  aws configure
   82  aws sts get-caller-identity
   83  ls
   84  terraform --ver
   85  terraform -version
   86  terrgrunt -version
   87  erragrunt -version
   88  terragrunt -version
   89  clear
   90  cd /d/awskr01
   91  pwd
   92  mkdir -p awskr01
   93  cd awskr01/
   94  mkdir -p infrastructure/modules/prepare
   95  mkdir -p infrastructure/live/000-prepare/ap-northeast-2
   96  git init
   97  git branch -m master main
   98  ls
   99  cd /d/awskr01
  100  ls
  101  mv awskr01/infrastructure .
  102  ls
  103  rm -rf awskr01
  104  ls
  105  ls infrastructure/
  106  mkdir -p infrastructure/modules/aws-network-spoke
  107  mkdir -p infrastructure/live/020-spokes/ap-northeast-2/network
  108  ls infrastructure/modules
  109  ls infrastructure/live
  110  ls
  111  cd infrastructure/
  112  ls
  113  cd modules/
  114  ls
  115  cd aws-network-spoke/
  116  ls
  117  touch infrastructure/modules/aws-network-spoke/variables.tf
  118  touch variables.tf
  119  touch main.tf
  120  touch outputs.tf
  121  ls
  122  cd ..
  123  ls
  124  cd ..
  125  ls
  126  cd ..
  127  ls
  128  cd infrastructure/
  129  ls
  130  cd live/
  131  ls
  132  pwd
  133  vi terragrunt.hcl
  134  vi terragrunt.hcl
  135  cd ..
  136  ls
  137  cd modules/
  138  ls
  139  cd prepare/
  140  ls
  141  vi main.tf
  142  ls
  143  cd ..
  144  ls
  145  cd ..
  146  ls
  147  cd live/
  148  ls
  149  cd 000-prepare/
  150  ls
  151  cd ap-northeast-2/
  152  ls
  153  vi terragrunt.hcl
  154  terragrunt init
  155  terragrunt apply
  156  history
  157  terragrunt destroy
  158  terragrunt force-unlock 80e4f17d-aebc-39fe-57d1-f5ce1f38ca76
  159  terragrunt apply
  160  ls
  161  cd ..
  162  ls
  163  cd ..
  164  ls
  165  cd ..
  166  ls
  167  cd modules/
  168  ls
  169  cd aws-network-spoke/
  170  ls
  171  vi var
  172  ls
  173  cd variables.tf
  174  vi variables.tf
  175  nano variables.tf
  176  ls\
  177  cd
  178  ls
  179  cd
  180  cd d/
  181  cd /d
  182  ls
  183  cd awskr01/
  184  ls
  185  cd infrastructure/
  186  ls
  187  cd modules/
  188  ls
  189  cd ..
  190  kls
  191  ls
  192  cd live/
  193  ls
  194  cd 020-spokes/
  195  ls
  196  cd ap-northeast-2/
  197  ls
  198  cd ..
  199  ls
  200  cd ..
  201  cd ..
  202  ls
  203  cd modules/
  204  ls
  205  cd aws-network-spoke/
  206  ls
  207  nano main.tf
  208  cd ..
  209  ls
  210  cd ..
  211  ls
  212  cd live/
  213  ls
  214  cd 020
  215  cd 020-spokes/
  216  ls
  217  cd ap-northeast-2/
  218  ls
  219  cd network/
  220  ls
  221  ls
  222  vi terragrunt.hcl
  223  cd ..
  224  ls
  225  cd ..
  226  ls
  227  cd ..
  228  ls
  229  cd ..
  230  ls
  231  ls
  232  cd live/
  233  ls
  234  cd 020-spokes/
  235  ls
  236  cd ap-northeast-2/
  237  ls
  238  cd network/
  239  ls
  240  vi terragrunt.hcl
  241  ls
  242   terragrunt destroy -auto-approve
  243  terragrunt destroy -auto-approve
  244  history
  245  cd ..
  246  ls
  247  cd ..
  248  ls
  249  cd .
  250  cd ..
  251  ls
  252  cd 000-prepare/
  253  ls
  254  cd ap-northeast-2/
  255  ls
  256  terragrunt destroy -auto-approve
  257  cd ..
  258  ls
  259  cd ..
  260  ls
  261  cd ..
  262  ls
  263  cd ..
  264  ls
  265  pwd
  266  ls -al
  267  git add .
  268  git commit -m "feat: 온프레미스(10.10) 기준 글로벌 네트워크 설계 및 다중 가용영역 기반 하이브리드 인프라 구축 완료"
  269  git push origin main
  270  history

