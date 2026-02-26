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


## 3-0. 사전체크
```
# eks 클러스터 만들기
## 0. 사전체크

# aws 로그인 확인
aws sts get-caller-identity

# 기본 리전 확인
aws configure list
```

### EKS 비용 주의 : 실습 끝나면 당일 Destroy
+ NAT Gateway
+ EC2 2대
+ EIP
+ LoadBalancer 생성 추가 과금

## 3-1. EKS 클러스터 생성 (eksctl) <- EKS Managed NodeGroup은 프리티어로 설치 할 수 없어서 삭제

(0) eksctl 설치 : t3.medium 사용 <- 프리티어로 설치 불가해서 삭제
15-20분 소요 : CloudFOrmation 스택 자동 생성됨
```
## 1. EKS 클러스터 생성 (eksctl) # t3.medium사용

eksctl create cluster \
--name megacluster \
--region ap-northeast-2 \
--nodegroup-name mega-nodegrp \
--node-type t3.medium \
--nodes 2 \
--nodes-min 1 \
--nodes-max 3 \
--managed
```

```
# 프리티어로 설치 할 수 없어서 삭제
# 클러스터 완전 삭제
eksctl delete cluster --name megacluster --region ap-northeast-2
```

(0) 안전한 버전으로 다시 eksctl 설치 전 삭제 잘 했는지 확인

### 설치 전 점검
+ CloudFormation에서 스택 0개인지 확인
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/769ff0e4-1c0a-42cd-ac87-dc3e7c099aa6" />
+ EC2 인스턴스 0개인지 확인
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/fbf07264-2670-4d50-854d-8ef91e9f1409" />

+ NAT Gateway 0개인지 CLI로 확인
콘솔에서는 상태가 Deleted로 떠야 함
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/048271d9-6976-4185-ab3d-c89115731250" />

CLi로 확인 시 State: deleted로 떠야 함
```
# 정상 삭제 예시
ubuntu@ip-172-31-8-161:~$ aws ec2 describe-nat-gateways --region ap-northeast-2 --query "NatGateways[*].{ID:NatGatewayId,State:State}"
[
    {
        "ID": "nat-0db9af5f8dd349ab7",
        "State": "deleted"
    }
]
```

+ Elastic IP 0개인지 확인 : []면 정상
```
aws ec2 describe-addresses \
--region ap-northeast-2 \
--query "Addresses[*].PublicIp"
```

(1) 안전한 버전으로 다시 eksctl 클러스터 설치 : t3.small
```
# 안전한 버전으로 eksctl 클러스터 다시 만들기
eksctl create cluster --name megacluster --region ap-northeast-2 --nodegroup-name mega-nodegrp --node-type t3.small --nodes 1 --managed

# 정상 작동 확인
k get nodes -o wide
```
+ READY 뜨면 정상
```
  NAME                                                STATUS   ROLES    AGE     VERSION               INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                        KERNEL-VERSION                   CONTAINER-RUNTIME
ip-192-168-31-241.ap-northeast-2.compute.internal   Ready    <none>   3m24s   v1.34.4-eks-efcacff   192.168.31.241   3.35.132.11   Amazon Linux 2023.10.20260216   6.12.68-92.122.amzn2023.x86_64   containerd://2.1.5
```
(2) Nginx 배포
```
# Nginx 배포
kubectl create deployment nginx-deploy --image=nginx --replicas=2
# 배포 성공 메시지 deployment.apps/nginx-deploy created

# 확인
k get pods -o wide
```
+ pods 확인 결과
```
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE                                                NOMINATED NODE   READINESS GATES
nginx-deploy-6f47956ff4-k8snt   1/1     Running   0          16s   192.168.27.181   ip-192-168-31-241.ap-northeast-2.compute.internal   <none>           <none>
nginx-deploy-6f47956ff4-l5gjx   1/1     Running   0          16s   192.168.12.86    ip-192-168-31-241.ap-northeast-2.compute.internal   <none>           <none>
```

```
# LoadBalancer Service 생성
kubectl expose deployment nginx-deploy --port=80 --type=LoadBalancer
service/nginx-deploy exposed

# 확인
k get svc
```
+ svc 확인 결과
```
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP                                                                  PORT(S)        AGE
kubernetes     ClusterIP      10.100.0.1      <none>                                                                       443/TCP        15m
nginx-deploy   LoadBalancer   10.100.18.115   a758158e8bc2447b6beda5091a60bdbf-81816330.ap-northeast-2.elb.amazonaws.com   80:30859/TCP   12s
```
(3) 브라우저 접속 확인
```
# 
nslookup a758158e8bc2447b6beda5091a60bdbf-81816330.ap-northeast-2.elb.amazonaws.com

```
### Nginx 기본 화면 나오면 성공
+ Name으로  : http://a758158e8bc2447b6beda5091a60bdbf-81816330.ap-northeast-2.elb.amazonaws.com
<img width="854" height="314" alt="image" src="https://github.com/user-attachments/assets/542a727a-49c0-405f-9524-8db73e603e82" />

+ Address로 접속확인 : nslookup a758158e8bc2447b6beda5091a60bdbf-81816330.ap-northeast-2.elb.amazonaws.com 명령으로 나온 주소로 확인
+ 3.37.109.17
<img width="644" height="311" alt="image" src="https://github.com/user-attachments/assets/5a3098f2-0a07-4e2d-93bd-2cbdf6a0abab" />

+ 43.203.88.34/
<img width="645" height="310" alt="image" src="https://github.com/user-attachments/assets/02491f25-f6d5-40b4-a076-5c4ea5513e5b" />

```
ubuntu@ip-172-31-8-161:~$ nslookup a758158e8bc2447b6beda5091a60bdbf-81816330.ap-northeast-2.elb.amazonaws.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	a758158e8bc2447b6beda5091a60bdbf-81816330.ap-northeast-2.elb.amazonaws.com
Address: 43.203.88.34
Name:	a758158e8bc2447b6beda5091a60bdbf-81816330.ap-northeast-2.elb.amazonaws.com
Address: 3.37.109.17
```
## 4. 자동완성기능 설정
```
# 자동 완성 설정
source <(kubectl completion bash)
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
```
## 5. 과금되는 리소스 삭제
```
# 지금 남아있는 것 확인
k get svc
k get deploy
k get pods

# nginx 서비스 삭제 (ELB 제거)
k delete svc nginx-deploy
# deployment 삭제
k delete deploy nginx-deploy

# EKS 클러스터 삭제
eksctl delete cluster --name megacluster --region ap-northeast-2

# EC2 인스턴스 확인
# running/stopped ,이름 태그, bastion 인지 확인

aws ec2 describe-instances \
--region ap-northeast-2 \
--instance-ids i-03a95f1e575a1991f i-0674604c909068c3f \
--query "Reservations[*].Instances[*].{ID:InstanceId,State:State.Name,Name:Tags[?Key=='Name']|[0].Value,Type:InstanceType}"

# ruuning 중인bastion 종료 시키기
# bastion 완전 종료 -> 바로 연결 끊김
aws ec2 terminate-instances \
--region ap-northeast-2 \
--instance-ids i-03a95f1e575a1991f
```
+ ssh연결 끊는 메시지 뜨면 성공
```
The system will power off now!

Connection closing...Socket close.

Connection closed by foreign host.

Disconnected from remote host(aws-bastion-0226) at 17:22:00.
```
+ 연결 바로 끊김
<img width="760" height="893" alt="image" src="https://github.com/user-attachments/assets/3c95f8fd-8325-46eb-b798-9aff7727e206" />

+ 이제 서버가 없으니까 AWS CloudShell이나 로컬 PC에서 AWS CLI 실행해서 과금 상태 확인
<img width="776" height="720" alt="image" src="https://github.com/user-attachments/assets/47149fbf-a172-468a-8cdf-e01cf0384d6a" />

```
# EC2 인스턴스 확인
aws ec2 describe-instances \
> --region ap-northeast-2 \
> --query "Reservations[*].Instances[*].{ID:InstanceId,State:State.Name}"

# 결과 모두 terminated 뜨면 과금 없음
[
    [
        {
            "ID": "i-03a95f1e575a1991f",
            "State": "terminated"
        }
    ],
    [
        {
            "ID": "i-0674604c909068c3f",
            "State": "terminated"
        }
    ]
]

# NAT Gatewasy 확인
aws ec2 describe-nat-gateways \
> --region ap-northeast-2 \
> --query "NatGateways[*].State"

# 결과 : deleted 뜨면 과금 없음
[
    "deleted"
]

# Load Balancer 확인
aws elbv2 describe-load-balancers \
> --region ap-northeast-2 \
> --query "LoadBalancers[*].State.Code"

# 결과 : []로 비어있으면 과금 없음
[]
# 한번 더 확인
aws elb describe-load-balancers --region ap-northeast-2

# 결과 : []로 비어있으면 과금 없음
{
    "LoadBalancerDescriptions": []
}

# Elastic IP 확인
aws ec2 describe-addresses \
> --region ap-northeast-2 \
> --query "Addresses[*].PublicIp"

# 결과 : []로 나오면 과금 없음
[]
```
+ 콘솔에서 직접 확인
(1) EC2 모두 인스턴트 상태 : 종료됨
<img width="1328" height="270" alt="image" src="https://github.com/user-attachments/assets/a6444b71-352b-4afc-835b-546303cfad37" />

(2) Nat Gateway 확인 (가장 중요)
<img width="1059" height="186" alt="image" src="https://github.com/user-attachments/assets/39683d2e-c554-40f1-b2b3-d6bea8eed3db" />

(3) Load Balancer 확인
<img width="1334" height="601" alt="image" src="https://github.com/user-attachments/assets/9b09deb8-bd06-46f1-8847-096db1e91fa0" />

(4) Elastic IP 확인
<img width="1331" height="285" alt="image" src="https://github.com/user-attachments/assets/1d7321e8-d04c-4c7d-99a2-66ae0517238d" />

(5) EKS 확인
<img width="1330" height="417" alt="image" src="https://github.com/user-attachments/assets/7b61a4cc-8ee1-4f29-901d-937ea4727fcb" />

=============
내일 실습 준비

1. 키페어 생성 : 0227-aws-bastion-key
<img width="368" height="142" alt="image" src="https://github.com/user-attachments/assets/96ad0854-2d1b-49a9-995d-1c0a367d5dec" />

콘솔에서 EC2 → 키 페어 → “키 페어 생성”

이름: 0227-bastion-key

키 페어 유형: RSA

프라이빗 키 형식: .pem

생성하면 .pem 파일 바로 다운로드 B1-MAIN/.ssh에 저장
<img width="368" height="142" alt="image" src="https://github.com/user-attachments/assets/938b3803-068d-4fb2-98b4-b058b474fcb1" />

2. Bastion EC2 생성 (t3.micro)

EC2 → 인스턴스 → 인스턴스 시작

기본 설정

이름: 0227-bastion

AMI: Ubuntu 22.04 (권장)

인스턴스 유형: t3.micro

키 페어

방금 만든 0227-bastion-key 선택

네트워크

퍼블릭 IP 자동 할당: 활성화

보안 그룹:

SSH (22)

소스: 내 IP (My IP 선택)

스토리지

기본 8GB 유지

→ 인스턴스 시작

3. 퍼블릭 IP 확인

EC2 → 인스턴스 → 0227-bastion 선택

Public IPv4 주소 복사

4. SSH 접속 (Xshell)
+ Host: 퍼블릭 IP
+ User: ubuntu
+ Authentication: Private Key
+ Key 파일: 0227-bastion-key.pem
+ 접속 성공하면 `whoami` ubuntu 나오면 성공
<img width="969" height="1032" alt="image" src="https://github.com/user-attachments/assets/b8ad93e7-a0fe-4ab3-bce4-2106f663d220" />

인스턴스 상태가 실행중이면 성공
<img width="1229" height="340" alt="image" src="https://github.com/user-attachments/assets/f482bb1f-c011-4f7c-b70c-78a942364041" />
+ public Ipv4 복사해서 ssh
