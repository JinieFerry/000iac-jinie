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
