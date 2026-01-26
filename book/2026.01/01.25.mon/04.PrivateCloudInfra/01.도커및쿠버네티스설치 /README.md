```
# 1. 시스템 패키지 업데이트
sudo apt update

# 2. 필수 패키지 설치
sudo apt install -y ca-certificates curl gnupg lsb-release
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f8091a78-baf2-4a53-8971-cadaea5513fd" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c86bc35d-7e9a-486e-9ff6-1e65e5799e4c" />

```
# 1. GPG 키 저장을 위한 디렉토리 생성 및 키 추가
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 2. 도커 저장소(Repository) 등록
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/3c27b024-5813-4f02-a0e8-d48e468caf34" />

3. 저장소 설정이 완료되면 실제 도커 엔진과 관련 도구들을 설치합니다.
```
# 저장소 정보 갱신
sudo apt update

# 도커 엔진, CLI, 컨테이너디, 플러그인 설치
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d4c9e3d8-d545-4caf-a4f8-c4403958188a" />

4.설치가 완료되었다면 서비스를 확인하고 테스트용 컨테이너를 실행해 봅니다.
```
#서비스 상태 확인:
sudo systemctl status docker
#Active: active (running) 상태가 보이면 정상입니다.

#Hello-World 테스트:
sudo docker run hello-world
#"Hello from Docker!"라는 메시지가 출력되면 도커가 이미지를 풀(Pull)하고 컨테이너를 실행하는 모든 과정이 정상적으로 작동하는 것입니다.
```

5.매번 sudo를 붙이는 것이 번거롭다면 현재 사용자를 docker 그룹에 추가하세요.
```
# 현재 사용자를 docker 그룹에 추가
sudo usermod -aG docker $USER

# 변경 사항 적용 (로그아웃 후 다시 로그인하거나 아래 명령 실행)
newgrp docker
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/888d10a4-740e-47eb-bd55-92e87bca0373" />

6. 브라우저 열어서 서비스 확인
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/5e32af09-e3ed-421c-a445-198c4f2cfd7a" />


