### 온프레미스 iptables NAT와 AWS NAT Gateway는 존재 목적이 다르다.

온프레미스에서는 라우터가 곧 NAT 장비가 될 수 있다.    
리눅스 서버에서 `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`로 내부 사설 Ip를 공인 IP로 변환해서 그냥 서버 한 대가 NAT역할을 수행 할 수 있기 때문이다.    
따라서,  IGW  같은 개념이 없어 직접 제어 할 수 있다.

AWS는 네트워크 구조가 다르다.   
+ AWS의 기본 전제
 + VPC는 완전 격리된 가상 네트워크
 + 인터넷은 IGW를 통해서만 나감
 + Public / Private subnet 개념 존재

<img width="912" height="586" alt="image" src="https://github.com/user-attachments/assets/9e506dc1-367e-435f-a7ad-0193239d1672" />
들어온 IP를 기억한다 = 들어온 헤더를 기억했다가 허락
그래서 ACL작성 시 '이것 외에 모두 허용하라'는 명령을 포함시킴
별도로 물리적 포트를 만들어 엄격하게 특별한 서비스만 허용하는 것 -> aws는 퍼블릭 IP가 많기 때                                                                                                                                                                                                                                                                                                                                                                                 문에 VPC 엔드포인트로 인터넷을 타지않고 QWS서비스를 프라이빗하게 연동하는 핵심 기술이다.
! s3는 vpc에 들어가지 않는다 ! aws로
ip가 아님 name으로 통신함 = 앞에 region과 age도 있는데 왜 유일해야 할까? 하나로 묶었기 때문이라고 추측    

+ peering 설명의 핵심
서로 다른 vpc 두개를 연결함, 건너뛰어서 통신하지는 못 함 (비용 큼) -> 허브엔스포크

VPN: Virtual Private Network    
외부에서 사설망을 접근 할 수 있도록 = 가상 터널링   
외붇에서 못 나가고 못 들어오도록

+ EKS 애서 다른 것
 + CIDIR

    192.168.10.x로 다 같은 IP 가상으로 묶었기 때문에 : 서비스를 여럿이서 하기 위해 a,b,c,d 를 하나로 묶은 : 노드의 개념이 사라졌음 = 소프트웨어 젓으로 밖에 묶을 수가 없음 
    Pod들은 10.x.x 혹은 172.x.
   awsd는 기에,노드 단위로 나누지만 파드들을 늘리는 개념에 가끼움

# 실습
### vpc 만들기 : 15분 동안 <- vpc 생성은 과금 되지 않음
krs vpc
krp 
apne

terragrunt 깔린 곳 밑에 디렉토리 새로 생성 (오늘 실습용)

~/tg/terragrunt.hcl (provider있는 것)걸 이동? k8s랑 같이 있는 게 실습에 편할 것? -> /tg밑에 vpcex디렉토리 만들기
1. 서울
network cidir: 10.200.0.0/16 = vpc 전체    
서브넷: 10.200.10 , 20 , 30, 40   
aws에 올리고 단톡방에 올린 후   
destroy   

2. 버지니아
이름은 약자로 줄여서
network cidir: 10.210.0.0/16
subnet 10.210.10 , 20 , 30 , 40

3. 유럽
