
<img width="682" height="196" alt="image" src="https://github.com/user-attachments/assets/5dca8d21-9f07-44af-8662-a285bd896103" />
컨트롤 플레인 안에도 큐블렛이 있음컨트롤 플레인 안에도 큐블렛이 있음. 큐블렛 빼고 다 파드임. 파드는 os에서 돌아야 함. 큐블렛이 os에서 파드를 만들어 둠. 파드 안에 컨테이너디를 집어넣음=이미지 집어넣음: 컨테이너라는 말을 잘 안 씀 이제.   
etd는 읽는 것과 쓰는 것 모두 api를 통해서 함. 덮어쓰지 않고 추가함 돌아갈 수 있게    


스케줄러는 파드 요청이 올 때 어느 노드에 배정할지 정함 : 요청 올 때 배정함    


큐블렛: 자기 노드 현상황을 계속 보고 하는 게 주된 일   


프록시: 제일 신기함. 유저들이 바로 접점을 가질 수 있는 곳. IP를 설정하는 일은 안 함
(파드들은 각자 아이피를 가지고 있음: 파드들에 대해 DHCP역할을 함:이건 CNI가 함:가상 네트워크카드)
, 자기 클러스터 안에 있는 모든 파드들의 IP를 알고, 수백개가 있어도 각 수백개에 들어있는 각 파드들의 아이피를 다 갖고 있는 것이 특징임:느려질 수 있음    
실무에서는 파드 볼 일이 거의 없음: 한 개 여도 디플로이컨트로 함    

### 실습1 : 하지않기로 함
0. 새로 디렉토리 만들기: exec_lab 3k8s_install도 라벨링 맞춰서 바꿈(k8s_lab)
1. 내 서버에서 NginX Deployment 2~3~5개: Nodeport 8080,30080 Pod 서비스
2. 웹 브라우저 접속 확인 http://192.168.115.251:30080
3. index.htmml "내 ip는 ~.x.x.x입니다." :자기가 하고 싶은 거 하기 
4. pod 중 1개만 index.html 주입 : 두 개 중 하나만 3이 들어가고, 나머지 하나에는 원래의 엔진엑스 디폴트가 보일 것
5. 접속
```
$ kubectl exec -it <pod네임>/bin/shell #alpine이라 bin 밑에 shell임
# cp index.html/usr/share/nginx/index.html
```
+ 새로고침 하면 Pod IP 바뀌도록 
+ 파이썬 웹 3.9-slim
+ 노드포트로 서비스 : 80아니고 30000~32767중 포트: ufw allow 포트넘버/tcp 로 파이어 월 개방해야함
<img width="1003" height="702" alt="image" src="https://github.com/user-attachments/assets/7b841b50-a3f7-43db-8a77-5771fc9e3a5b" />

<img width="1199" height="785" alt="image" src="https://github.com/user-attachments/assets/dfedecfb-b19b-40bb-b80b-4e1f172b1618" />

업그레이드 버전
<img width="1376" height="790" alt="image" src="https://github.com/user-attachments/assets/3f294710-f1b8-4dd1-a940-c12f97e9ed30" />
<img width="1376" height="796" alt="image" src="https://github.com/user-attachments/assets/6fca39fb-850e-43bc-957f-aab23bc8dc6e" />

### 실습 1 <- 할 수 있는 개 수 까지만 (2-3개)
목적: 디플로이먼트 5개가 유지 되도록   
디플로이먼트와 node포트를 실습하는 것   
index.html을 바꿔서 ~~를 실감하기, 주입하기: container가 웹서비스를 하고 있는데 exec로 주입할 수 있음: 들어가서 index.html으로 "여기는 ~~페이지입니다"라고 웹에 띄울 화면 내용을 바꿔서 웹으로 접속해서 확인하기.     
로드발란싱은 proxy가 한다는 것을 체감하면 됨   

### 리마인드 실습 : 자주 쓰는 명령어

```
#vmmaster1에서

$ k cluster-info # 잘 돌고 있는지 확인

$ k get node # AGE는 나이: shutdown 안 한 만큼의 시간
```

===
Jinie Log
===

##### 실습참고 원본
2026.02.03.Tue 17:28 ~ 짧게 실습 연습

https://github.com/putto4u/04.PrivateCloudInfra/blob/main/21.exec/010.%20%EC%97%94%EC%A7%84x%20%EC%95%BC%EB%AF%88%EB%A1%9C%20%EB%B0%B0%ED%8F%AC1.md

04.PrivateCloudInfra/21.exec/010. 엔진x 야믈로 배포1.md
=> nginx web 말고 python 3.9-slim으로 배포 수정해서 하기

#### 디렉토리 정리(vmmaster1)
+ 
+ k8s_install -> k8s_lab 으로 수정
```
# mv옵션 원래디렉토리명 바꾸고 싶은 디렉토리명
$ mv k8s_install k8s_lab

# 디렉토리로 이동
$ cd k8s_lab
$ ls

# 실습 별 디렉토리 분리

# 이전의 k8s_install 디렉토리로 진행한 실습
# 새 디렉토리00_cluster_setup 생성
$ mkdir -p 00_cluster_setup

# k8s_install 디렉토리 안에 있던 실습 파일 모두 00_cluster_setup으로 이동
$ mv hosts.ini k8s_reset.yaml k8s_setup_playbook.yaml kubeadm-init.yaml 00_cluster_setup/

# 확인
$ ls

# 2026.02.03.화 디플로이 실습 디렉토리 생성
$ mkdir -p 01_yaml_nginx
```
<img width="922" height="86" alt="image" src="https://github.com/user-attachments/assets/0744060d-4216-43c7-a9e5-44bfaa8871df" />
<img width="909" height="139" alt="image" src="https://github.com/user-attachments/assets/5eb213ab-9181-48a7-8588-c3eb368ecd13" />
<img width="925" height="212" alt="image" src="https://github.com/user-attachments/assets/d9e97dd2-9ace-4360-b29b-8707bf7f3db1" />

#### 디플로이 실습
```
# 디렉토리 구조 확인
$ tree ~/k8s_lab
```

# 디플로이 실습 디렉토리로 이동
$ cd 
```

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
$ k get node -o wide # 더 길게 나옴
```
결과: 강사님이랑 똑같은 상태
vm1: not reade
vmmaster1 ready
<img width="575" height="101" alt="image" src="https://github.com/user-attachments/assets/a6697ee6-0875-47b2-9623-bcdfa8e5ce41" />
```
$ k get pod
$ k get service
$ kubectl apply -f app.yaml #-f:파일명 옵션
$ kubectl describe pod #전체파드 #뒤에 특정파드 붙여서 짧게도 볼 수 있음
$ kubectl exec -it my-web /bin/sh
```

내일은 pvc 볼륨 하나 만들어서 할 것
