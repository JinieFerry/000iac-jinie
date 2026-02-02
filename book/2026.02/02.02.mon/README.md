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

