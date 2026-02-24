# log4hum-history.md
Project: awskr01
Author: JinieFerry
Environment: Windows 10 / Git Bash (MINGW64)
AWS Account: 219850556765 (IAM User: Ferry)
Region: ap-northeast-2

---

## Phase 0 – Tool Installation & Verification

choco install terraform -y
terraform --version
terragrunt --version
aws --version

---

## Phase 1 – GitHub Repository Setup

cd /d
mkdir awskr01
cd awskr01
git init
git branch -M main
git remote add origin https://github.com/JinieFerry/awskr01.git
git pull origin main
git remote -v

---

## Phase 2 – AWS CLI Configuration

aws configure
aws sts get-caller-identity

---

## Phase 3 – Infrastructure Directory Structure Creation

cd /d/awskr01

mkdir -p infrastructure/modules/prepare
mkdir -p infrastructure/modules/aws-network-spoke
mkdir -p infrastructure/live/000-prepare/ap-northeast-2
mkdir -p infrastructure/live/020-spokes/ap-northeast-2/network

---

## Phase 4 – Root Terragrunt Configuration

File:
infrastructure/live/terragrunt.hcl

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

Resources:
- aws_ecr_repository.backend_repo
- aws_ecr_repository.frontend_repo
- aws_iam_role.eks_cluster_role
- aws_iam_role_policy_attachment.eks_cluster_policy

---

## Phase 6 – Prepare Execution Layer 작성

File:
infrastructure/live/000-prepare/ap-northeast-2/terragrunt.hcl

Execution:

cd infrastructure/live/000-prepare/ap-northeast-2
terragrunt init
terragrunt apply

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

terragrunt force-unlock 80e4f17d-aebc-39fe-57d1-f5ce1f38ca76
yes

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

cd infrastructure/live/020-spokes/ap-northeast-2/network
terragrunt destroy -auto-approve

---

## Phase 13 – Prepare Destroy

cd infrastructure/live/000-prepare/ap-northeast-2
terragrunt destroy -auto-approve

---

## Phase 14 – Git Commit

cd /d/awskr01
git add .
git commit -m "feat: 온프레미스(10.10) 기준 글로벌 네트워크 설계 및 다중 가용영역 기반 하이브리드 인프라 구축 완료"
git push origin main

---

## End State

- Prepare resources destroyed
- Network skeleton ready
- Backend S3 + DynamoDB 유지
- GitHub main branch 최신 상태

```
user@DESKTOP-5AFO9PS MINGW64 /d/awskr01 (main)
$ history
    1  chooco --version
    2  choco --version
    3  choco install terraform -y
    4  terraform -version
    5  terraform --version
    6  terragrunt --version
    7  clear
    8  brew tap hashicorp/tap
    9  brew tap hashicorp/tap
   10  brew tap hashicorp/tap
   11  clear
   12  Get-ExecutionPolicy
   13  cd ..
   14  ls
   15  ls
   16  clear
   17  cd /d
   18  mkdir awskr03
   19  cd awskr03/
   20  pwd
   21  git init
   22  git remote add origin https://github.com/JinieFerry/awskr03.git
   23  git remote -v
   24  git pull orgin main
   25  git pull origin main
   26  clear
   27  cd /d
   28  mkdir awskr01
   29  cd awskr01
   30  pwd
   31  git init
   32  git remote add origin https://github.com/JinieFerry/awskr01.git
   33  git branch -M main
   34  git pull origin main
   35  tree
   36  ls
   37  git remote -v
   38  cd ~
   39  pwd
   40  cd /d/awskr01
   41  pwd
   42  vi terragrunt.hcl
   43  ls
   44  vi .gitignore
   45  vi .gitignore
   46  vi .gitignore
   47  aws --version
   48  git add .gitignore
   49  git commit -m "Update gitignore for Terraform security"
   50  git config --global user.name "JinieFerry"
   51  git config --global user.email "jinie.sej@gmail.com"
   52  git commit -m "Update gitignore for Terraform security"
   53  git commit -m "Update gitignore for Terraform security"
   54  mv /c/Users/B1-MAIN/.pem/aws-bastion-key.pem /c/Users/user/.ssh/
   55  ls /c/Users/B1-MAIN/.pem
   56  ls /c/Users/B1-MAIN/.pem
   57  ls /c/Users/user/Downloads
   58  ls /c/Users/user/Downloads
   59  ls /c/Users/user/Downloads
   60  
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
```
