## vscode 설치 + github랑 연동하기 (github repositories 패키지, korean 패키지 설치하기)
+ jinieferry와 연동함
## 테라폼 설치
https://putto4u.github.io/05.PublicCloud/130.%20AWS%EC%9E%90%EB%8F%99%ED%99%94%EC%99%80%20%ED%85%8C%EB%9D%BC%ED%8F%BC%EC%84%A4%EC%B9%98/903.%EC%B4%88%EC%BD%94%EB%A0%88%ED%8B%B0%EB%A1%9C%20%ED%85%8C%EB%9D%BC%ED%8F%BC%20%EC%84%A4%EC%B9%98.html
+ 파워쉘 혹은 gitbash에서 꼭 관리자권한으로 실행하기
:윈도우에서 만든 apt 같은 것

1. Chocolatey 설치
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

<img width="707" height="593" alt="image" src="https://github.com/user-attachments/assets/2db49abe-32fd-470a-a0cb-d9513a917eaa" />
+ 파워쉘에서 복사붙여넣기가 안될 시 상단바 우클릭 > 속성 > Crtl+Shift+C/V를 복사붙여넣기로 사용에 체크하고 확인
2. Chocolatey 설치 확인
```Git bash
#관리자 모드로 실행해서 초콜레티 버전 확인
choco --version
```
<img width="581" height="370" alt="image" src="https://github.com/user-attachments/assets/9d4e04ea-c92f-4f5f-910a-3c8c55d89ecb" />

3. Terraform 설치
이제 Chocolatey를 이용해 Terraform을 간단하게 설치합니다.
```Git Bash
# Admin 권한 권장
choco install terraform -y
```

테라폼은 프로바이더의 역할이 중간에 반드시 껴야 함 (번역) API로 번역함 

4. 설치 검증
Terraform 명령어가 정상적으로 실행되는지 최종 확인합니다.

```Git Bash
terraform -version
```


   # 2026.02.19.thu #
   ## 05.150.0010 ##
   # ~테라폼을 이용한 Nginx Deployment 구축~
```
   #again with jd
#쿠버네티스 정상 동작 확인: Ready 떠야 함
   k get nodes

#kubeconfig 존재 확인
# server: https://192.168.115.251:6443 이렇게 VM IP면 정상
   cat ~/.kube/config

#테라폼 설치 확인
   terraform -version
#테라폼 없으면 만들기
#sudo snap install terraform --classic
   ls

#작업 디렉토리 생성
   mkdir -p terraform-k8s-nginx
   cd terraform-k8s-nginx
   touch provider.tf main.tf variables.tf outputs.tf
   ls
```
+ provider.tf 작성
```
   nano provider.tf
```
+ povider.tf
```
provider "kubernetes" {
  config_path = "~/.kube/config"
}
```
+ variables.tf 작성
```
   nano variables.tf
```
+ variables.tf 
```
variable "nginx_labels" {
  description = "Nginx 리소스에 적용할 라벨"
  type        = map(string)
  default = {
    app  = "nginx"
    tier = "frontend"
  }
}

variable "replica_count" {
  description = "생성할 파드 개수"
  type        = number
  default     = 3
}

```
+ main.tf 작성 (Deployment + Service)
```
   nano main.tf
```
+ main.tf
```
resource "kubernetes_deployment" "nginx_deploy" {

  metadata {
    name   = "nginx-deployment"
    labels = var.nginx_labels
  }

  spec {
    replicas = var.replica_count

    selector {
      match_labels = var.nginx_labels
    }

    template {

      metadata {
        labels = var.nginx_labels
      }

      spec {
        container {
          name  = "nginx"
          image = "nginx:1.21"

          port {
            container_port = 80
          }

          resources {
            limits = {
              cpu    = "500m"
              memory = "512Mi"
            }

            requests = {
              cpu    = "250m"
              memory = "256Mi"
            }
          }
        }
      }
    }
  }
}

# service 추가
resource "kubernetes_service" "nginx_svc" {

  metadata {
    name = "nginx-service"
  }

  spec {
    selector = var.nginx_labels

    port {
      port        = 80
      target_port = 80
    }

    type = "NodePort"
  }
}
```
+ outputs.tf 작성
 ```
   nano outputs.tf
```
+ outputs.tf
```
output "node_port" {
  value = kubernetes_service.nginx_svc.spec.0.port.0.node_port
}
```
+ 테라폼 실행 순서
 ```
# 초기화 
   terraform init
# 계획 확인
   terraform plan
# 정상결과
#kubernetes_deployment.nginx_deploy will be created
#kubernetes_service.nginx_svc will be created
#
#Plan: 2 to add, 0 to change, 0 to destroy.

# 배포
   terraform apply -auto-approve

# 배포 확인
   k get pods
   k get svc
   k get pods -l app=nginx #192.168.115.251:30954
   kubectl get deploy
   kubectl get pods | grep nginx
```
+ 브라우저 접속 성공: 192.168.115.251:30954
<img width="913" height="1033" alt="image" src="https://github.com/user-attachments/assets/2919bea1-ed54-441c-a098-b3a03058a673" />


