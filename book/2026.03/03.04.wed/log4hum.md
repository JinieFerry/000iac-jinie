# 2026.03.04.wed

## 강사님 버전 : 창을 열어두고 왔다갔다 새 탭을 여러개 열어둬야 함 순서가 섞여있음 (비추천)

1. 아시아 태평양 (서울) vpc 생성 : SPK

(1) SPK-01   
+ 가용영역 2
+ 퍼블릭 서브넷 0    
+ 프라이빗 서브넷 2    
+ 서브넷 (퍼블릿은 0, 아닌 것은 3 혹은 4를 줌)     
  + 10.10.10.0/20    
  + 10.10.20.0/20    
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/6bc5b834-3058-4ce1-b462-2a19b533c914" />
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/d8893562-6125-4373-a6d1-cad208258697" />

+ 
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/350b204c-9518-4c09-b43c-9ef22165afc6" />
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/6884331c-6856-4afc-9dd3-9afaf0ee9857" />

(2) 라우팅 테이블
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c3a78103-0322-47da-a892-a63a97cdafdf" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/aaaa240f-3a94-4663-95bf-27b68c72bcd0" />

(3) 라우팅 테이블 추가 : 0.0.0.0 -> TGW 설정
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2e3c7786-0b66-4418-b10b-5bf5a1565c9c" />

(4) TGW Gateway  추가
+ ECMP:
+ 교차계정 공유 옵션 : 회사에서 많이 사용함
+ 아이피를 주진 않지만 CIDR 블록은 줌 : 트렌짓 게이트웨이랑 옛날 장비를 써서 CISCO 라우터 장비들은 자기가 갖고 있는 포트에 상대방 대역의 아이피가 있어야 하기 때문에 이때 아이피 대역을 상대방과 맞춰줄 때 (대기업)
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/1bf7ed4d-3d73-4c28-93df-89e8ebbc138d" />
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/a7d78462-691e-46eb-ae6a-c121784df4d7" />
+ ARN : Amazon Resource Name
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/0d58fe76-0cb3-4052-956e-ac7479e36305" />
+ 상태 : Pending -> Available 시간 좀 걸림
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/962cb3c8-9b93-4e03-a4cc-150959982145" />

(5) Trasnsit Gat
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/abb53cdb-3ce1-4244-8bf7-53b7d08bd6ea" />
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/d9f8179a-f002-4193-aa36-e0e64bb74b39" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/84393c8e-3575-4e87-bd16-e9142e4498e9" />
-------
# 클린 버전

```
[SPK01-vpc]          [VPCHUB-vpc]
10.10.0.0/16    ←→   10.0.0.0/16
      \               /
       Transit Gateway (TGWKR)
```
## 1. SPK01 VPC 생성     
VPC → VPC 생성     
항목값생성할 리소스VPC 등이름SPK-01IPv4 CIDR10.10.0.0/16가용영역 수2퍼블릭 서브넷 수0 ← 먼저 0으로 설정!프라이빗 서브넷 수2프라이빗 서브넷 CIDR 110.10.10.0/24프라이빗 서브넷 CIDR 210.10.20.0/24NAT 게이트웨이없음VPC 엔드포인트없음   

+ VPC 생성 클릭

## 2. VPCHUB VPC 생성
VPC → VPC 생성    
항목값생성할 리소스VPC 등이름VPCHUBIPv4 CIDR10.0.0.0/16가용영역 수2퍼블릭 서브넷 수2프라이빗 서브넷 수2
+ 10.0.10.0/24
+ 10.0.20.0/24
+ 10.0.30.0/24
+ 10.0.40.0/24

NAT 게이트웨이없음VPC 엔드포인트없음   
+ VPC 생성 클릭

## 3. Transit Gateway 생성   
VPC → Transit Gateway → Transit Gateway 생성    
항목값이름TGWKRASN비워두기 (기본값)DNS 지원 VPN ECMP 지원 기본 라우팅 테이블 연결 기본 라우팅 테이블 전파      
+ 생성 클릭 → 잠깐 기다리기 (Available 될 때까지)    

## 4. Transit Gateway Attachment 2개 생성
VPC → Transit Gateway 연결 → Attachment 생성    
첫 번째 (SPK01 연결)    
항목값이름TGW-SPK01Transit Gateway IDTGWKR 선택연결 유형VPCVPC IDSPK01-vpc 선택서브넷2a, 2b 둘 다 체크    
+ 생성
두 번째 (HUB 연결)    
항목값이름TGW-HUBTransit Gateway IDTGWKR 선택연결 유형VPCVPC IDVPCHUB-vpc 선택서브넷2a, 2b 둘 다 체크    
+ 생성 → 둘 다 Available 될 때까지 기다리기    

## 5. 라우팅 테이블 편집   
SPK01 라우팅 테이블   
VPC → 라우팅 테이블 → SPK01-rtb-private1 선택 → 라우팅 편집    
DestinationTarget0.0.0.0/0Transit Gateway → TGWKR    
+ 저장 → SPK01-rtb-private2도 똑같이!     
VPCHUB 라우팅 테이블    
VPCHUB-rtb-private1 선택 → 서브넷 연결 편집     

+ VPCHUB-subnet-private1 체크
+ VPCHUB-subnet-private2 체크

+ 저장
## 01. VPC 2개 생성 : 아시아 태평양 (서울)   
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/d85a6948-7e6c-48f5-b284-6f47c3f864a3" />
 />
 <img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/bcd6e24f-757d-4b84-aae2-14540462740a" />
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/f5cf6a99-11f6-40a3-8ebc-1f64a5242837" />

