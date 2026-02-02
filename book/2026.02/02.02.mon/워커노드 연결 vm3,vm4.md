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

0. vm3,vm4 설정
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



1. vmmaster1의 공개키 복사: id_ed25519.pub
```
#vmmaste1에서 이동
$ cd ~/.ssh
$ ls
$ cat id_ed25519.pub #공개키 복사
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEAJtJP2fyKnKlifzo6EByU99AwY611T8OebgYMTA+IG master@vmmaster1

2.vm3와 vm4에게 vmmaster1의 공개키 넣어주기
```
# vm3과 vm4에서 이동
$ cd ~/.ssh
$ ls
$ sudo nano authorized_keys #여기에 vmamster1의 공개키 전체 붙여넣기 (공백없이) 

```

3. vmmaster1에서 vm3,vm4 접속 확인
```
#vm3
$ ssh master@vm3 # yes
# vm4
$ ssh master@vm4 # yes
```


4. k8s_reset.yaml 파일 만들기: 클러스터 초기화 해야 할 때
```
# vmmaster1에서
$ cd k8s_install
$ nano k8s_reset.yaml
```

##### k8s_reset.yaml
```
---
- name: Kubernetes 클러스터 환경 강제 초기화
  hosts: all
  become: yes
  tasks:
    - name: kubeconfig 존재 여부 확인
      stat:
        path: /etc/kubernetes/admin.conf
      register: kube_config

    - name: 클러스터 정보 강제 삭제 (멈춤 방지 로직 적용)
      shell: |
        # 1. 서비스 중지 및 관련 프로세스 강제 종료
        systemctl stop kubelet || true
        killall -9 kubelet kube-proxy 2>/dev/null || true
        
        # 2. 런타임 내 실행 중인 모든 컨테이너 중지
        crictl ps -q | xargs -r crictl stop || true
        
        # 3. 마운트된 볼륨 해제 (Lazy unmount로 프로세스 점유 무관하게 해제)
        df | grep /var/lib/kubelet | awk '{print $6}' | xargs -r umount -l || true
        
        # 4. kubeadm 초기화 실행
        kubeadm reset -f
        
        # 5. CNI 설정 및 잔여 데이터 완전 삭제
        rm -rf /etc/cni/net.d /var/lib/kubelet/* /etc/kubernetes/*
        ip link delete cni0 || true
        ip link delete flannel.1 || true
        
        # 6. iptables 규칙 초기화 (네트워크 꼬임 방지 핵심)
        iptables -F && iptables -t nat -F && iptables -X
      when: kube_config.stat.exists
      async: 120  # 대규모 볼륨 해제 시 멈춤 방지를 위해 비동기 처리
      poll: 5
```
5-0. -K 안먹음: yes 입력 : 아마도 master passwd가 1234여서 보안 정책이 강해져서 막히는 듯 (bad password issue) 에러
+ master passwd, root passwd 모두 8자리로 바꾸기 ㅣ 똑같이 K8s 소스 리스트 추가 및 패키지 설치에서 막힘
+ fstab # 한개만 남기고 제거하기

+ 패스워드 12345678로 모두 통일
```
# vmmaster1,vm1~3 모두에서 실행: 12345678로 통일
$ sudo passwd master #ansible로 실행하는 BECOME패스워드가 이거랑 같음
$ sudo passwd root 
```

+ fstab 중복 # 제거
```
#vmmaster1에서
$ cd /etc
$ sudo vi fstab # 중복된 # 위에서 x로 지우기
# 저장하고 나가기 : esc
$ :wq!
```

+ apt 업데이트
```
$ sudo apt update
$ sudo apt upgrade
```
  
5. vmmaster1 k8s_setup_playbook.yaml파일 전체를 02.02 버전으로 수정하기
```
# vi로 열어야 명령어 단축키 옵션 사용 가능
$ vi k8s_setup_playbook.yaml
# esc누르고 dG:전체 줄 삭제 2yy: 2줄 복사, p: 붙여넣기로 vm3,vm4 추가하기
$ 192.168.115.3 vm3
$ 192.168.115.4 vm4

#esc: 저장하고 나오기
$ wq!
```

#### 깨끗하게 노드 유지하기
k8s설치+워커노드 연결 후에 마스터랑 워커노드 스냅샷 찍은 후에 파일/ media/vdi 불변으로 변경하기
<img width="1269" height="706" alt="image" src="https://github.com/user-attachments/assets/77b6d554-16cd-46cf-ac8e-10aa45f6756d" />
