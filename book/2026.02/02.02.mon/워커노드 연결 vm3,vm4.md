##### Nginx 서비스 종료: fastAPI 설치하기

k8s를 워커노드로 등록한 만큼 노드가 늘어남: join 안 하면 노드 x

전산실의 클러스터: 효율성을 위해 개념이 확장됨     
금융권은 클러스터의 개념이 달라짐(전통적인 방식) 서버를 잡은 곳 까지만 클러스터 : 아직 실무에 많이 남아있음.   
새로 만들어서 이전하지 않는 이상 그대로 사용   

api가 모든 신호를 받아-> etcd에 저장  

etcd는 리눅스의 etc와 같은 개념(d:daemon)    
:healthy 신호가 안오면 api가 etcd에 저장    

scheduler가 배정    

프록시가 자기 아이피와 파드들의 아이피를 알고 있음: 파드들이 어느 노드에 들어가있는지 알아야 함     
컨트롤 플레인은 유저 서비스에는 관여하지 않음   

파드는 여러개의 컨테니어를 묶은 것. Docker가 실행하던 것은 Containerd가 실햄(도커도 nerd받아서 함 원래)   
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/619465b2-ac64-4035-a968-c143410ede70" />

RestAPI는 yaml,json등 과 같이 표현방식을 정해서 주는 것   
옛날엔 한 페이지가 동적 스크린 처럼 보이게 하려면 java script를 사용했음   
자바 스크립트를 서버단과 프론트에서도 했었음 -> REST API 테이블로 하는 걸로 바뀜 (키:값으로 자바스크립트 알아서 짜서 쓰면 됨)   

기존의 데몬은 다 listen 모드: 해당 포트를 listen -> api server를 watch    
CNI와 TOKEN에서 에러가 제일 많이 났음    
CNI는 가상의 프로그램으로 만든 네트워크 카드:Container Network Interface    
Flannel 부터 시작함 작은 규모 부터 하려고   

마스터 노드 'k8s 설정 완료'로 스냅샷 
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/1c1b5b29-17de-4c85-b4cf-dd593a36badf" />
<img width="962" height="692" alt="image" src="https://github.com/user-attachments/assets/60acdca6-9302-41f7-8af7-c050f989b347" />

### vm 3,4 추가

워커 노드 2개 더 만들기

#### 1. k8s 설치 완료 전의 스냅샷에서 복제해서 vm3,vm4 생성
cpu: 1개
시스템 / 마더보드/ Base memory: 1000gb로 (마스터는 4000), UEFI 체크
hdd: 10g
<img width="955" height="692" alt="image" src="https://github.com/user-attachments/assets/414ebb3e-ddb9-4ed2-a9bb-35e80b34806f" />
Mac address 새로고침
<img width="802" height="512" alt="image" src="https://github.com/user-attachments/assets/e7279fde-aef8-4ef9-a544-353a52c46b34" />


#### 2. hostname/hosts/고정 IP/SSH 키 넘기기: master노드에서 넘어갈 수 있게

```
$ cd /etc/hosts
```
127.0.1 localhost
127.1.1 vm3 vm4
gw: 127.10.1
vm3,vm4 자기거 빼고 마스터 등록하고

#### 3. yaml로 두 대 동시에 k8s 설치하기  
error: key, cni 에서 에러 많이 날 것 -> reset

#### 4. gpt 내가 원하는 yaml 생성

#### 5.Token: join할 때 필요

master의 .ssh에 저장해둔 ssh 키: .pub으로 연결    
=> master에서 모든 다른 노드로 접속할 수 있게 연결: 마스터노드, 워커노드 둘 다 한테 줘야 함

----
Jinie Log
----

1. vm3,vm4 설정
```
# 아이피 확인
$ ip a

# vm3 아이피 할당: 192.168.115.3/16
# vm4 아이피 할당: 192.168.115.4/16
# 01-netcfg.yaml인지  50-cloud-init.yaml인지 확인

$ cd /etc/netplan
$ ls # 여기에 있는 yaml 파일을 다음에서 수정

# 수정
$ sudo vi /etc/netplan/50-cloud-init.yaml
# addresses:vm3의 고정 아이피 주소 = 192.168.115.3/16 으로 수정
# addresses:vm4의 고정 아이피 주소 = 192.168.115.4/16 으로 수정
# via:게이트 웨이 = 192.168.10.1

# 저장하고 나가기
$ wq!

# v3고정아이피와 게이트웨이 적용
$ sudo netplan apply

# 확인
$ ip a
# enp0s3 , 192.168.115.3/16 , 192.168.115.4/16
$ ping -c 2 8.8.8.8

```
<img width="806" height="570" alt="image" src="https://github.com/user-attachments/assets/8aa8a790-d804-4099-9919-5763f34ebfe8" />
<img width="811" height="580" alt="image" src="https://github.com/user-attachments/assets/1420d221-6274-49fe-9c39-2f8c337c28bc" />



2. vmmaster1의 공개키 복사: id_ed25519.pub
```
#vmmaste1에서 이동
$ cd ~/.ssh
$ ls
$ cat id_ed25519.pub #공개키 복사
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEAJtJP2fyKnKlifzo6EByU99AwY611T8OebgYMTA+IG master@vmmaster1

3.vm3와 vm4에게 vmmaster1의 공개키 넣어주기
```
# vm3과 vm4에서 이동
$ cd ~/.ssh
$ ls
$ sudo nano authorized_keys #여기에 vmamster1의 공개키 전체 붙여넣기 (공백없이) 

```

4. vmmaster1에서 vm3,vm4 접속 확인
```
#vm3
$ ssh master@vm3 # yes
# vm4
$ ssh master@vm4 # yes
```
