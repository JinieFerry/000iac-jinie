# 2026.03.04.wed
+ 강사님 버전-> onbo.md : 창을 열어두고 왔다갔다 새 탭을 여러개 열어둬야 함 순서가 섞여있음 (비추천)
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
+ 이름 : 0304-HUB-KR-01
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
<img width="1598" height="808" alt="image" src="https://github.com/user-attachments/assets/61bf262e-6f07-4d49-b2b8-d6a0a4fa35c9" />
<img width="1283" height="842" alt="image" src="https://github.com/user-attachments/assets/3d674e1e-27a0-49b2-a541-ee7a6937c7c8" />
<img width="1283" height="832" alt="image" src="https://github.com/user-attachments/assets/4a038cfd-fe7b-4e00-b2ce-874233b57019" />
<img width="1307" height="788" alt="image" src="https://github.com/user-attachments/assets/ca8a5396-db14-4943-b0b4-060fcbf5645b" />

## 3. Transit Gateway 생성   
**VPC → Transit Gateway → Transit Gateway 생성**      
+ 이름,설명: 0304-TGW-KR-01    
+ ASN 비워두기(기본값)
+ DNS 지원 VPN
+ ECMP 지원 기본
+ 라우팅 테이블 연결 기본
+ 라우팅 테이블 전파      
+ 생성 클릭 → 잠깐 기다리기 (Available 될 때까지)
<img width="1228" height="851" alt="image" src="https://github.com/user-attachments/assets/d39f5563-30e1-4318-9cc6-b03761066765" />
<img width="1232" height="342" alt="image" src="https://github.com/user-attachments/assets/82a8a93f-071b-4560-87ab-acf887885b0e" />
<img width="1225" height="552" alt="image" src="https://github.com/user-attachments/assets/efdadf0c-9929-4021-884a-7cff9969f452" />

## 4. Transit Gateway Attachment 2개 생성
**VPC → Transit Gateway 연결 → Attachment 생성**      
(1) 첫 번째 Attachment 생성 (0304-TGW-KR-01 연결)
+ 이름: 0304-TGW-SPK-01
+ Transit Gateway ID: 0304-TGW-KR-01 선택
+ 연결 유형 VPC
+ VPC ID : 0304-SPK-KR-01 선택
+ 서브넷 2 a, 2b 둘 다 체크    
+ 생성
<img width="987" height="550" alt="image" src="https://github.com/user-attachments/assets/4ecfc288-2fb3-4fe8-ac68-8e291bf676fc" />
<img width="660" height="263" alt="image" src="https://github.com/user-attachments/assets/9ad8e82a-3988-4700-860a-69b4f19dd405" />
<img width="658" height="262" alt="image" src="https://github.com/user-attachments/assets/8162e950-7f68-4185-9fff-825918cf36c5" />
<img width="642" height="424" alt="image" src="https://github.com/user-attachments/assets/e7f60180-7ec3-4cc6-8515-c84bec322136" />
<img width="1222" height="841" alt="image" src="https://github.com/user-attachments/assets/62393d82-977e-48d6-9ba7-a9d15ce05828" />
<img width="1222" height="487" alt="image" src="https://github.com/user-attachments/assets/c7db9dcc-8142-430e-8228-29338dbf4aac" />

(2) 두 번째 Attachment 생성 (HUB 연결)    
+ 이름: 0304-TGW-HUB-01
+ Transit Gateway ID : 0304-TGW-KR-01 선택
+ 연결 유형 VPC
+ VPC ID : 0304-HUB-KR-01 선택
+ 서브넷 2 a, 2b 둘 다 프라이빗 체크    
+ 생성 → 둘 다 Available 될 때까지 기다리기    
<img width="658" height="265" alt="image" src="https://github.com/user-attachments/assets/d5cbe91e-527b-4a87-8f03-ac22acdd04b6" />
<img width="656" height="258" alt="image" src="https://github.com/user-attachments/assets/27102702-4b53-4cee-8b5e-52fe992a3502" />
<img width="516" height="188" alt="image" src="https://github.com/user-attachments/assets/001943ed-b518-475a-a846-660276843fd6" />
<img width="531" height="400" alt="image" src="https://github.com/user-attachments/assets/5d81f284-ecda-4fcb-bcb8-e6427b4f5b15" />
<img width="523" height="402" alt="image" src="https://github.com/user-attachments/assets/bc287525-4cfc-454d-b878-867155ab7eef" />
<img width="802" height="505" alt="image" src="https://github.com/user-attachments/assets/5ec7b62f-b255-4a4a-90a9-f000fae9c91b" />

## 5. 라우팅 테이블 편집   
**VPC → 라우팅 테이블 → SPK01-rtb-private1 선택 → 라우팅 편집**    
(1) 허브 private1 편집
+ 0304-HUB-KR-01-rtb-private1-ap-northeast-2a 선택
+ 라우팅 편집 클릭
+ 라우팅 추가 클릭
+ Destination : 0.0.0.0/16
+ Target : Transit Gateway → 0304-TGW-KR-01
+ 변경사항 저장
<img width="989" height="405" alt="image" src="https://github.com/user-attachments/assets/0cf31dc7-8ec6-4b2b-8f56-d96508c0aadb" />
<img width="992" height="263" alt="image" src="https://github.com/user-attachments/assets/614a41b0-9465-4bdd-8b86-c1b4b77cf8e1" />
<img width="804" height="445" alt="image" src="https://github.com/user-attachments/assets/daf370b2-6d71-46a1-95a3-a6bff56d5521" />


(2) 허브 private2 편집
+ 0304-HUB-KR-01-rtb-private2-ap-northeast-2b 선택     
+ 라우팅 편집 클릭
+ 라우팅 추가 클릭
+ Destination : 0.0.0.0/16
+ Target : Transit Gateway → 0304-TGW-KR-01
+ 변경사항 저장
<img width="991" height="372" alt="image" src="https://github.com/user-attachments/assets/233589f8-c474-41a3-9051-6ef7183f00b5" />
<img width="988" height="313" alt="image" src="https://github.com/user-attachments/assets/76f48aba-81f2-417e-9f93-ca165b3a9eba" />
<img width="804" height="451" alt="image" src="https://github.com/user-attachments/assets/3bceb017-15e4-467c-9cce-182028c39946" />

  


