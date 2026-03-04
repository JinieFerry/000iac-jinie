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
**VPC → VPC 생성**         
+ 이름: 0304-SPK-KR-01
+ IPv4
+ CIDR : 10.10.0.0/16
+ 가용영역 수 2
+ 퍼블릭 서브넷 수 0 ← 먼저 0으로 설정!
+ 프라이빗 서브넷 수 2
+ 프라이빗 서브넷 CIDR 110.10.10.0/24
+ 프라이빗 서브넷 CIDR 210.10.20.0/24
+ NAT 게이트웨이없음VPC 엔드포인트없음
+ VPC 생성 클릭
<img width="1277" height="839" alt="image" src="https://github.com/user-attachments/assets/39f461c3-92f5-46aa-af25-e841278be1f8" />
<img width="1256" height="828" alt="image" src="https://github.com/user-attachments/assets/d32ba199-f17a-4138-9db3-2a317f690009" />
<img width="1636" height="819" alt="image" src="https://github.com/user-attachments/assets/c42ff1b5-fb96-46d6-93bf-c87fb6aba091" />


## 2. VPCHUB VPC 생성
**VPC → VPC 생성**    
+ 이름 : 0304
+ VPCHUBIPv4 CIDR10.0.0.0/16
+ 가용영역 수 2
+ 퍼블릭 서브넷 수 2
+ 프라이빗 서브넷 수 2
+ 10.0.10.0/24
+ 10.0.20.0/24
+ 10.0.30.0/24
+ 10.0.40.0/24
+ NAT 게이트웨이없음VPC 엔드포인트없음   
+ VPC 생성 클릭
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/40e3cf22-7e00-454a-8dbd-73dd26c62e08" />
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/8b6129cd-5bc4-4b3d-9914-287ce98c5db4" />
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/0024850e-a767-4a0f-9f4c-1fd370fc0f46" />

## 3. Transit Gateway 생성   
**VPC → Transit Gateway → Transit Gateway 생성**      
+ 이름,설명: 0304-TGW     
+ ASN 비워두기(기본값)
+ DNS 지원 VPN
+ ECMP 지원 기본
+ 라우팅 테이블 연결 기본
+ 라우팅 테이블 전파      
+ 생성 클릭 → 잠깐 기다리기 (Available 될 때까지)
<img width="999" height="1046" alt="image" src="https://github.com/user-attachments/assets/5aea5c10-e732-48af-b27a-28c317884e5c" />
<img width="999" height="1046" alt="image" src="https://github.com/user-attachments/assets/92b3e85f-0e87-48a3-a5bb-ec394b9cc9b1" />
<img width="999" height="1046" alt="image" src="https://github.com/user-attachments/assets/072afcab-d8c0-4d2b-b8cc-52357c233513" />

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


