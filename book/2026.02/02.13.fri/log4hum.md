# 사전 준비 #
## vm15-01 ~ vm-04 ssh키 패스워드 없이 자동 로그인 : ssh키 등록하기 ##
### 목표: 모두 password 없이 들어가게 만들기.
### 숙제`: VM1 APT MySQL 설치
```
내 PC (Xshell 키)
        ↓
   vmmaster1
        ↓
   vm15-01 ~ 04
```
+ 이슈: ansible, terraform, kubeadm join 자동화, 노드 추가 실습 할 때마다 계속 비번 물어봄 → 자동화가 깨짐
+ 해결: vmmaster1에 vm15-01~04의 ssh키 등록하기 
4096의 x2로 배수로 늘리기 마더보드의 베이스메모리
+ 가변으로 설정했기 때문에 50으로 잡아도 50으로 안나옴
  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7304f580-f414-4481-92df-be3550b4b5e0" />
```
sudo vi /var/lib/kubelet/
ls
vi config.yaml 
#수정해야 함
```
/memory로 검색, n을 치면 그 다음 검색으로 넘어감
192.168.115.251:30000/dashboards로 그라파나 브라우저 접속 > search/'import dash board'/1860 / import

<img width="975" height="1014" alt="image" src="https://github.com/user-attachments/assets/041bc6a2-58c5-45f3-a883-e5f7b8fd4739" />
정보/저장소의 이름을 보고 늘려야 함

#### vmmaster1
+ 스냅샷 지우기 (모두 다 지우기)
+ 프로메테우스 설치 전에 vm1,2,3,4를 켜고 해야 다 깔림
