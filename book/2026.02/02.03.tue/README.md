
<img width="682" height="196" alt="image" src="https://github.com/user-attachments/assets/5dca8d21-9f07-44af-8662-a285bd896103" />
컨트롤 플레인 안에도 큐블렛이 있음컨트롤 플레인 안에도 큐블렛이 있음. 큐블렛 빼고 다 파드임. 파드는 os에서 돌아야 함. 큐블렛이 os에서 파드를 만들어 둠. 파드 안에 컨테이너디를 집어넣음=이미지 집어넣음: 컨테이너라는 말을 잘 안 씀 이제.   
etd는 읽는 것과 쓰는 것 모두 api를 통해서 함. 덮어쓰지 않고 추가함 돌아갈 수 있게    


스케줄러는 파드 요청이 올 때 어느 노드에 배정할지 정함 : 요청 올 때 배정함    


큐블렛: 자기 노드 현상황을 계속 보고 하는 게 주된 일   


프록시: 제일 신기함. 유저들이 바로 접점을 가질 수 있는 곳. IP를 설정하는 일은 안 함
(파드들은 각자 아이피를 가지고 있음: 파드들에 대해 DHCP역할을 함:이건 CNI가 함:가상 네트워크카드)
, 자기 클러스터 안에 있는 모든 파드들의 IP를 알고, 수백개가 있어도 각 수백개에 들어있는 각 파드들의 아이피를 다 갖고 있는 것이 특징임:느려질 수 있음    
실무에서는 파드 볼 일이 거의 없음: 한 개 여도 디플로이컨트로 함    

### 실습1
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

내일은 pvc 볼륨 하나 만들어서 할 것
