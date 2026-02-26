# 2026.02.26.thu

# Bastion 서버 만들기 

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
   19  # kubectl 설치
   20  curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.34.2/2025-11-13/bin/linux/amd64/kubectl
   21  chmod +x kubectl
   22  mkdir -p $HOME/bin
   23  mv kubectl $HOME/bin/
   24  echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
   25  source ~/.bashrc
   26  # 확인
   27  kubectl version --client

   28  # 단축어 설정
   29  alias k='kubectl'
   33  echo "alias k='kubectl'" >> ~/.bashrc
   34  source ~/.bashrc

   36  # 단축어 적용
   37  source ~/.bashrc
   38  # 단축어 확인
   39  k version --client
# 버전 나오면 성공
```

## 3. eksctl설치
```
   40  # eksctl 설치
   41  ARCH=amd64
   42  PLATFORM=$(uname -s)_$ARCH
   43  curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_${PLATFORM}.tar.gz"
   44  tar -xzf eksctl_${PLATFORM}.tar.gz -C /tmp
   45  sudo install -m 0755 /tmp/eksctl /usr/local/bin

   46  # eksctl 설치 확인
   47  ekstctl version
   48  eksctl version
```
## 4. 도커 설치
```
   51  ## 4. 도커 설치

   52  # 기존 도커 제거
   53  sudo apt-get remove docker docker-engine docker.io containerd runc

   54  # 필수 패키지 설치
   55  sudo apt-get update
   56  sudo apt-get install -y ca-certificates curl gnupg

   58  # 도커 공식 GPG 키 등록
   59  sudo install -m 0755 -d /etc/apt/keyrings
   60  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   61  sudo chmod a+r /etc/apt/keyrings/docker.gpg

   62  # 공식 리포지토리 추가
   63  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
