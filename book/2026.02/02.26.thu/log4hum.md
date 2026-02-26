# 2026.02.26.thu

# Bastion 서버 만들기 (05.250.0010/0015/0025)

<img width="527" height="909" alt="image" src="https://github.com/user-attachments/assets/59795b7c-765d-4fd5-91e7-a9eb35ea866f" />


## 1. EC2 만들기


## 1-1. key pair 생성
<img width="1081" height="1033" alt="image" src="https://github.com/user-attachments/assets/094da2d3-a005-4225-9396-6dc7c911d8a8" />


## 1-2. 보안그룹 생성
<img width="1081" height="1033" alt="image" src="https://github.com/user-attachments/assets/41bf1f33-781b-492a-bf4d-c79cd0e73e5e" />

  
## 1-3. EC2 생성
<img width="1314" height="427" alt="image" src="https://github.com/user-attachments/assets/2d01e331-ccb6-4215-bbc5-7f0616453f89" />

### (0) Xshell 접속 터미널 만들기
<img width="734" height="649" alt="image" src="https://github.com/user-attachments/assets/d58d504b-cc53-489b-9b91-355079749024" />
<img width="489" height="401" alt="image" src="https://github.com/user-attachments/assets/9521364f-38b3-4c43-a21f-b74c51c2127e" />
<img width="734" height="649" alt="image" src="https://github.com/user-attachments/assets/b588b56d-9e27-486a-937a-fc141f955862" />
<img width="412" height="225" alt="image" src="https://github.com/user-attachments/assets/dede0931-bd88-4a66-9b91-8e6dad6fa40b" />
<img width="770" height="1038" alt="image" src="https://github.com/user-attachments/assets/46615730-cdd2-4793-a1fa-6b0229d4d870" />

### (1) ssh 접속 성공
### (2) aws cli 설치
  
```
  # 2026.02.26.thu #
  # 패키지 업데이트
  # sudo apt update
  sudo apt install -y unzip curl

  # aws cli 설치
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  unzip awscliv2.zip
  sudo ./aws/install

  # aws cli 설치 확인
  aws --version
  history
```


## 1-4. IAM 액세스 키 설정: aws 콘솔
<img width="753" height="1033" alt="image" src="https://github.com/user-attachments/assets/9686807b-95ef-44c5-b410-239efd15820d" />
<img width="753" height="1033" alt="image" src="https://github.com/user-attachments/assets/1cdf40f6-b246-426f-a7d1-2061fda864a9" />
<img width="753" height="1033" alt="image" src="https://github.com/user-attachments/assets/a0b575e5-17f3-404c-a4c3-08c292f83bd4" />
<img width="753" height="1033" alt="image" src="https://github.com/user-attachments/assets/eb9bae32-b752-4b6a-b546-0aa4d28006c5" />


### (1) 바로 액세스키와 비밀액세스 키 복사해두고 .csv 파일 다운로드 (나중에 다시 보거나 다운로드 할 수 없음)

<img width="415" height="175" alt="image" src="https://github.com/user-attachments/assets/721735ff-0b5e-4cbd-ad49-40455b258f64" />
<img width="947" height="634" alt="image" src="https://github.com/user-attachments/assets/6c520601-d52a-400d-9ea1-b8ebb0ec9ee5" />

### (2) csv 파일에서 액세스 아이디와 비밀 액세스 키 복사해서 입력
<img width="643" height="106" alt="image" src="https://github.com/user-attachments/assets/ec3c4b27-adb0-453c-93a2-da9a12fcd630" />
<img width="1200" height="700" alt="image" src="https://github.com/user-attachments/assets/91211303-1e3f-44b1-9f29-b8aa63ea9923" />

```
# aws configure
aws configure
# csv파일의 id와 secret access key 복사해서 작성
# AWS Access Key ID [None]: 깃허브에 못 올림
# AWS Secret Access Key [None]: Iamc6Kg7OWQZGr2LL/UUQ/38gMsebfCwJ5mXyZlV
# Default region name [None]: ap-northeast-2
# Default output format [None]: json

 # 확인
 aws sts get-caller-identity
```

### aws sts 확인 결과 : 성공 <- 따라치는 거 아님 예시로 보라고

```
{
    "UserId": "AIDATGMA5IVORRAHD4HCS",
    "Account": "219850556765",
    "Arn": "arn:aws:iam::219850556765:user/Ferry"
}
```


## 2. kubectl 설치

```
# kubectl 설치
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.34.2/2025-11-13/bin/linux/amd64/kubectl
chmod +x kubectl

mkdir -p $HOME/bin
mv kubectl $HOME/bin/
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
# 확인
kubectl version --client

# 단축어 설정
alias k='kubectl'
echo "alias k='kubectl'" >> ~/.bashrc
source ~/.bashrc

# 단축어 적용
source ~/.bashrc
# 단축어 확인
k version --client
# 버전 나오면 성공
```

## 3. eksctl설치

```
# eksctl 설치
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_${PLATFORM}.tar.gz"

tar -xzf eksctl_${PLATFORM}.tar.gz -C /tmp

sudo install -m 0755 /tmp/eksctl /usr/local/bin

# eksctl 설치 확인
ekstctl version
eksctl version
```

## 4. 도커 설치

```
# 기존 도커 제거
sudo apt-get remove docker docker-engine docker.io containerd runc

# 필수 패키지 설치
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# 도커 공식 GPG 키 등록
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 공식 리포지토리 추가
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 도커 엔진 설치
sudo apt-get update

# 설치 시간 소요
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# sudo 명령어 없이 작업 할 수 있도록 권한 설정
# 도커 그룹 생성
sudo groupadd docker
# 이미 전에 권한 설정해두면 메시지 뜸: groupadd: group 'docker' already exists
# 이미 이전 실습에서 했으면 메시지 떠도 정상

# 현재 로그인 한 사용자 $USER를 docker그룹에 추가
sudo usermod -aG docker $USER

# 변경된 그룹 권한을 현재 셀에 즉시 정용
newgrp docker

# 도커 엔진 확인 : sudo 안치고 실행되면 성공
docker run hello-world

# Hello from Docker! 뜨면 성공
# docker run [이미지 이름] [컨네이터 내부에서 실행할 명령]형식으로 작성해야 하니까 그냥 도커 공식 테스트 이미지 사용하기
```

## 5. Terraform 설치

```
# 5. terraform 설치
# 필수 패키지 설치
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common wget

# HashiCorp 공식 GPG 키 등록
 wget -O- https://apt.releases.hashicorp.com/gpg |

 # > 뜨면 아래 코드 입력
sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# 공식 리포지토리 추가
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \

# 이것도 > 뜨면 입력하면 됨
sudo tee /etc/apt/sources.list.d/hashicorp.list

# 테라폼 설치
sudo apt-get update
sudo apt-get install terraform

# 버전 확인
terraform -v

# 자동완성 설정
terraform -install-autocomplete
```

## 6. Terragrunt 설치

```
# Terragrunt 설치
export TG_VERSION="v0.55.0"

# 다운로드
wget "https://github.com/gruntwork-io/terragrunt/releases/download/${TG_VERSION}/terragrunt_linux_amd64"

# 실행권한 부여
chmod +x terragrunt_linux_amd64

# 시스템 경로로 이동
sudo mv terragrunt_linux_amd64 /usr/local/bin/terragrunt

# 설치 확인
terragrunt --version
```


### 1~6까지의 단계로 성공 시 상태: IaC + EKS + 컨테이어 제어 노드 완성      

```
# 최종 확인
which terraform
which terragrunt
which docker
which kubectl
```

전부 경로 /usr/local/bin 또는 /usr/bin으로 나오면 됨
```
# 예시
ubuntu@ip-172-31-8-161:~$ which terraform
/usr/bin/terraform
ubuntu@ip-172-31-8-161:~$ which terragrunt
/usr/local/bin/terragrunt
ubuntu@ip-172-31-8-161:~$ which docker
/usr/bin/docker
ubuntu@ip-172-31-8-161:~$ which kubectl
/home/ubuntu/bin/kubectl
```
AWS CLI ✔

kubectl ✔

eksctl ✔

Docker ✔

Terraform ✔

Terragrunt ✔

# 3. aws configure 액세스 키 연결하기 = eks 클러스터 만들기 (05.550.0020)


## 0. 사전체크
```
# eks 클러스터 만들기
## 0. 사전체크

# aws 로그인 확인
aws sts get-caller-identity

# 기본 리전 확인
aws configure list
```
