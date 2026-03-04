# 2026.03.04.wed
+ 강사님 버전-> onbo.md : 창을 열어두고 왔다갔다 새 탭을 여러개 열어둬야 함 순서가 섞여있음 (비추천)
-------
# 클린 버전

```
          HUB VPC
        10.0.0.0/16
      ┌──────────────┐
      │ Bastion      │
      │ 10.0.30.x    │
      └──────┬───────┘
             │
        Transit Gateway
             │
      ┌──────────────┐
      │ Nginx        │
      │ 10.10.20.x   │
      └──────────────┘
        SPOKE VPC
      10.10.0.0/16
```
## 1. SPK01 VPC 생성     
**VPC → VPC 생성**    
(1)vpc   
+ 이름: 0304-SPK-KR-01
+ IPv4
+ CIDR : 10.10.0.0/16
+ 가용 영역 수 2

(2)subnet  
+ 퍼블릭 서브넷 수 0 ← 먼저 0으로 설정!
+ 프라이빗 서브넷 수 2
+ 프라이빗 서브넷1 CIDR 10.10.10.0/24
+ 프라이빗 서브넷2 CIDR 10.10.20.0/24
+ NAT 게이트웨이없음 VPC 엔드포인트없음
+ VPC 생성 클릭
<img width="1277" height="839" alt="image" src="https://github.com/user-attachments/assets/39f461c3-92f5-46aa-af25-e841278be1f8" />
<img width="1256" height="828" alt="image" src="https://github.com/user-attachments/assets/d32ba199-f17a-4138-9db3-2a317f690009" />
<img width="1636" height="819" alt="image" src="https://github.com/user-attachments/assets/c42ff1b5-fb96-46d6-93bf-c87fb6aba091" />


## 2. HUB VPC 생성
**VPC → VPC 생성** 
(1) vpc
+ 이름 : 0304-HUB-KR-01
+ CIDR : 10.0.0.0/16
+ 가용 영역 수 2
(2) subnet
+ 퍼블릭 서브넷 수 2
+ 프라이빗 서브넷 수 2
+ public1 : 10.0.10.0/24
+ public2 : 10.0.20.0/24
+ private1 : 10.0.30.0/24
+ private2 : 10.0.40.0/24
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
+ DNS 지원
+ VPN ECMP 지원
+ 기본 라우팅 테이블 연결
+ 기본 라우팅 테이블 전파      
+ 생성 클릭 → 잠깐 기다리기 (Available 될 때까지)
<img width="1228" height="851" alt="image" src="https://github.com/user-attachments/assets/d39f5563-30e1-4318-9cc6-b03761066765" />
<img width="1232" height="342" alt="image" src="https://github.com/user-attachments/assets/82a8a93f-071b-4560-87ab-acf887885b0e" />
<img width="1225" height="552" alt="image" src="https://github.com/user-attachments/assets/efdadf0c-9929-4021-884a-7cff9969f452" />

## 4. Transit Gateway Attachment 2개 생성
**VPC → Transit Gateway 연결 → Attachment 생성**      
(1) Spoke 연결 Attachment 생성 (0304-TGW-KR-01 연결)
+ 이름 : 0304-TGW-SPK-01
+ Transit Gateway ID: 0304-TGW-KR-01
+ 연결 유형 VPC
+ VPC ID : 0304-SPK-KR-01 선택
+ 서브넷 2 : private1 , private2 
+ 생성
<img width="987" height="550" alt="image" src="https://github.com/user-attachments/assets/4ecfc288-2fb3-4fe8-ac68-8e291bf676fc" />
<img width="660" height="263" alt="image" src="https://github.com/user-attachments/assets/9ad8e82a-3988-4700-860a-69b4f19dd405" />
<img width="658" height="262" alt="image" src="https://github.com/user-attachments/assets/8162e950-7f68-4185-9fff-825918cf36c5" />
<img width="642" height="424" alt="image" src="https://github.com/user-attachments/assets/e7f60180-7ec3-4cc6-8515-c84bec322136" />
<img width="1222" height="841" alt="image" src="https://github.com/user-attachments/assets/62393d82-977e-48d6-9ba7-a9d15ce05828" />
<img width="1222" height="487" alt="image" src="https://github.com/user-attachments/assets/c7db9dcc-8142-430e-8228-29338dbf4aac" />

(2) Hub 연결  Attachment 생성   
+ 이름: 0304-TGW-HUB-01
+ Transit Gateway ID : 0304-TGW-KR-01 
+ 연결 유형 VPC
+ VPC ID : 0304-HUB-KR-01 
+ 서브넷 2 : private1 , private2    
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


(2) 허브 private2 편집
+ 0304-HUB-KR-01-rtb-private2-ap-northeast-2b 선택     
+ 라우팅 편집 클릭
+ 라우팅 추가 클릭


(3) spoke private1 편집
+  0304-SPK-KR-01-rtb-private1-ap-northeast-2a tjsxor
+  라우팅 편집 클릭
+  라우팅 추가 클릭
+ 10.10.0.0/16  0.0.0.0/0 (강사님은 0.0.0.0/16 추천 하지만 gpt는 잘못된 cidr라고 지적해서 다르게 진행함)
+ Target : Transit Gateway → 0304-TGW-KR-01
+ 변경사항 저장
<img width="989" height="405" alt="image" src="https://github.com/user-attachments/assets/0cf31dc7-8ec6-4b2b-8f56-d96508c0aadb" />
<img width="1467" height="307" alt="image" src="https://github.com/user-attachments/assets/6c778fa8-af41-4f4d-a41c-6273289e14da" />
<img width="1471" height="515" alt="image" src="https://github.com/user-attachments/assets/1b3cc7c6-9445-4d78-8e56-a7b08e3afe1a" />

(4) spoke private2 편집
+ 0304-SPK-KR-01-rtb-private2-ap-northeast-2b 선택
+ 라우팅 편집 클릭
+ 라우팅 추가 클릭
+ Destination : 0.0.0.0/0 (강사님은 0.0.0.0/16 추천 하지만 gpt는 잘못된 cidr라고 지적해서 다르게 진행함)
+ Target : Transit Gateway → 0304-TGW-KR-01
+ 변경사항 저장
<img width="991" height="372" alt="image" src="https://github.com/user-attachments/assets/233589f8-c474-41a3-9051-6ef7183f00b5" />
<img width="1472" height="304" alt="image" src="https://github.com/user-attachments/assets/cb5d2ccb-6ce0-4bb8-8abc-d46aebf43347" />
<img width="1470" height="521" alt="image" src="https://github.com/user-attachments/assets/112e9c0c-95c3-4acb-8570-b8c8bce22233" />

## 6. EC2 생성
**EC2 -> 인스턴스 시작**     
(1) 설정
+ Name : 0304-SPK-NGINX-01
+ 애플리케이션 및 OSI : Ubuntu
+ 인스턴스 : t3.micro
+ 키페어 : 0227-aws-bastion-key (기존 키 페어)
+ 네트워크 '편집' 클릭
  + VPC : 0304-SPK-KR-01
  + Subnet : private2 (10.10.20.0/24)
  + Public IP 자동할당 : Disable
+ 보안그룹 생성 : 0304-SPK-NGINX-SG-01
  + Inbound
   + SSH 22
   + source = 10.0.0.0/16

  + HTTP 80
   + source = 0.0.0.0/0
<img width="902" height="300" alt="image" src="https://github.com/user-attachments/assets/7bd874f4-d72d-4a98-bc84-b97e077e358c" />
<img width="878" height="716" alt="image" src="https://github.com/user-attachments/assets/afe18a64-d377-4004-b66a-0713adb7565b" />
<img width="1234" height="743" alt="image" src="https://github.com/user-attachments/assets/c648ffa3-e658-4b21-8eb7-07ece860dfd1" />
<img width="1220" height="731" alt="image" src="https://github.com/user-attachments/assets/74b401da-b14e-4efe-a0d3-466229809ef7" />
<img width="1218" height="838" alt="image" src="https://github.com/user-attachments/assets/dfe222e0-23bf-4489-a421-96377066758b" />
<img width="1301" height="828" alt="image" src="https://github.com/user-attachments/assets/cd5413b2-ae6a-4153-88ba-51842956c0db" />
<img width="1303" height="468" alt="image" src="https://github.com/user-attachments/assets/83a47fed-72af-4758-857a-4ffd9195f726" />
 
## 7. Bastion 서버 생성 (Hub)
**EC2 -> 인스턴스 시작**      
+ Name : 0304-HUB-BASTION-01
+ 애플리케이션 및 OSI : Ubuntu
+ 인스턴스 : t3.micro
+ 키페어 : 0227-aws-bastion-key (기존 키 페어)
+ 네트워크 '편집' 클릭
  + VPC : 0304-HUB-KR-01
  + Subnet : private1 (10.0.30.0/24)
+ Public IP : Disable
+ 보안그룹생성
  + 이름 : 0304-HUB-BASTION-SG-01
  + SSH : 22
  + source = 0.0.0.0/0
<img width="1302" height="852" alt="image" src="https://github.com/user-attachments/assets/399f0554-e352-49bf-8b02-394ee7a796d3" />
<img width="1046" height="770" alt="image" src="https://github.com/user-attachments/assets/b925cbf6-f500-4152-b904-d4e23d99fae0" />
<img width="1305" height="471" alt="image" src="https://github.com/user-attachments/assets/8cab01f3-6792-4a03-929c-337c86a8fba6" />
<img width="1306" height="839" alt="image" src="https://github.com/user-attachments/assets/6e015d86-0825-45c7-9d30-1e497087b8bc" />
<img width="1713" height="159" alt="image" src="https://github.com/user-attachments/assets/35d4abce-11a1-4485-b970-6c88a156684b" />


## 7. 엔드포인트 생성
(1) EC2 인스턴스 연결 엔드포인트 생성      
**VPC->PrivateLink 및 Lattice->엔드포인트**   
+ 엔드포인트 생성 클릭
+ 이름 : 0304-HUB-ICE-01
+ 유형 : EC2 인스턴스 연결 엔드포인트
+ 네트워크설정 : (0304-HUB-KR-01-vpc)
+ 보안그룹 : 0304-HUB-BASTION-SG-01
+ 서브넷 : 프라이빗1 (0302-HUB-KR-01-subnet-private-1-ap-northeast-2a)
+ IPv4
<img width="1919" height="205" alt="image" src="https://github.com/user-attachments/assets/589ae663-278a-4769-9de3-a3db51c0054e" />
<img width="1471" height="753" alt="image" src="https://github.com/user-attachments/assets/94db40b8-7c23-42dc-b524-a04db026afed" />
<img width="1460" height="487" alt="image" src="https://github.com/user-attachments/assets/8fe3d1c3-6aab-4701-b593-381e2572f484" />
+ 0304-HUB-ICE-01 상태가 사용가능으로 변해야 함 (30초 ~ 2분)
<img width="1696" height="195" alt="image" src="https://github.com/user-attachments/assets/8b45b228-8c27-4662-a70d-bb18a25b3836" />

## 8. Bastion 접속
**EC2 -> 인스턴스**        
(1) 0304-HUB-BASTION-01 선택 → 연결  
+ 연결유형 : 프라이빗 IP를 사용하여 연결 클릭
<img width="1913" height="244" alt="image" src="https://github.com/user-attachments/assets/536d5a11-170e-4ca6-8f1e-b1dc4410738f" />
+ 엔드포인트가 아직 생성 중이면 (create-in-progress)가 뜸 : 연결 버튼 활성화 될 때 까지 잠시 대기
<img width="1468" height="797" alt="image" src="https://github.com/user-attachments/assets/bd1bf018-966d-49d1-b230-697c908d5211" />
  + (create-complete) 후에 연결 버튼 활성화 됨 - 연결 버튼 클릭
<img width="366" height="74" alt="image" src="https://github.com/user-attachments/assets/b1ad3d0b-2665-4850-b554-41f8f1b5faef" />
<img width="1467" height="797" alt="image" src="https://github.com/user-attachments/assets/5fabafa3-6476-47d4-afe7-09a48f3ff4b8" />

+ 접속하면 ubuntu@ip-10-0-30-x으로 뜸
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/9b58ece5-75cb-425f-975f-810164e56be0" />

(2) SPK nginx 서버 IP 확인
+ 0227-bastion의 프라이빗 IPv4 주소 확인 : 172.31.1.181
<img width="1473" height="475" alt="image" src="https://github.com/user-attachments/assets/ddfd9527-989e-4e20-8abb-2358d0035243" />
+ 0304-SPK-NGINX-01의 프라이빗 IPv4 주소 확인 : 10.10.20.127
<img width="1466" height="526" alt="image" src="https://github.com/user-attachments/assets/8034c235-341d-409d-bdbf-96c562e7dcbc" />


## 8. Nginx 설치 : NAT 만든 후에 
**Bastion에서**
+ ssh ubuntu@10.10.20.x 접속 : 
+ 업데이트
```
sudo apt update
sudo apt install nginx -y
```
+ 확인
```
curl localhost
```
  + 정상 : Welcome to nginx

## 8. 네트워크 테스트
(1) Bastion에서 ping 테스트
```
# ping 10.10.20.x # 위에서 확인한 프라이빗 주소 넣기
ping 10.10.20.127
# 정상 : 64 bytes from
```
+ 결과 : `PING 10.10.20.127 ~ bytes of data`
<img width="1921" height="945" alt="image" src="https://github.com/user-attachments/assets/e8d4ce23-eb10-4de2-a86c-b332b4518560" />
+ 정상 결과 뜨면 종료 : Ctrl + C

(2) nginx ssh 접속 테스트
+ Permission denied (publickey) 방지
```
nano bastion.pem
# 직접 로컬 PC에 있는 0227-aws-bastion-key.pem 파일 전체 복사해서 붙여넣기

# 저장하고 나오기

# 권한 설정
chmod 400 bastion.pem
```

<img width="1669" height="912" alt="image" src="https://github.com/user-attachments/assets/881e24c2-f58c-43de-8115-20b5717b300e" />

+ nginx 서버 접속
```
ssh -i bastion.pem ubuntu@10.10.20.127
```
+ nginx 서버 접속 성공 : ubuntu@ip-10-10-20-127
<img width="1656" height="809" alt="image" src="https://github.com/user-attachments/assets/1cbee3c5-d7bd-4c93-ae26-491609dabd07" />

+ nginx 설치 : 지금 nginx 서버는  SPK Private Subnet 에 있기 때문에 SPK -> Internet 구조가 없다
  + 구조는 다음과 같다

  ```
     nginx
  10.10.20.127
       ↓
  10.10.20.1 (subnet gateway)
       ↓
  ! Internet 없음 !
  ```
(3) HUB NAT Gateway 통해서 인터넷 나가기
**VPC-> NAT 게이트웨이 -> NAT 게이트웨이 설정**
+ 이름 : 0304-HUB-NAT-01
+ 가용성 모드 " 영역별
+ 서브넷 : 0304-HUB-KR-01-subnet-public1
+ 탄력적 IP (Elastic IP) : 새로 할당
<img width="1809" height="870" alt="image" src="https://github.com/user-attachments/assets/f3cdc90d-daec-4c6a-bcd8-cce37562c714" />
<img width="1472" height="508" alt="image" src="https://github.com/user-attachments/assets/daada0af-f3e3-4168-be55-e195e5a20f18" />

**VPC -> 라우팅 테이블**
+ SPK 쪽은 라우팅 테이블 프라이빗에 NAT 추가
  + 0304-SPK-KR-01-rtb-private1-ap-northeast-2a
  + 0304-SPK-KR-01-rtb-private2-ap-northeast-2c
```
# 설치
sudo apt update
sudo apt install nginx -y

# 확인
niginx -v
```

## 9. HTTP 테스트
**Bastion에서**     
```
curl 10.10.20.x
```
+ 정상 : Welcome to nginx

여기까지 성공하면 통신 성공 , NAT는 과금방지로 아직 생성하지 않음
```
Hub
   Bastion
      │
      │ TGW
      │
Spoke
   Nginx
```
