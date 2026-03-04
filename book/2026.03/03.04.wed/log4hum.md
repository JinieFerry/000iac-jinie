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
+ 10.10.0.0/16 (강사님은 0.0.0.0/16 추천 하지만 gpt는 잘못된 cidr라고 지적함 우선 10.10으로 진행함)
+ Target : Transit Gateway → 0304-TGW-KR-01
+ 변경사항 저장
<img width="989" height="405" alt="image" src="https://github.com/user-attachments/assets/0cf31dc7-8ec6-4b2b-8f56-d96508c0aadb" />
<img width="908" height="642" alt="image" src="https://github.com/user-attachments/assets/683a656e-8866-4a52-8c4e-ec7f052ae14b" />
<img width="732" height="451" alt="image" src="https://github.com/user-attachments/assets/269d68d0-ba95-4bc3-ac96-96f80d029b4d" />


(2) 허브 private2 편집
+ 0304-HUB-KR-01-rtb-private2-ap-northeast-2b 선택     
+ 라우팅 편집 클릭
+ 라우팅 추가 클릭
+ Destination : 10.10.0.0/16 (강사님은 0.0.0.0/16 추천 하지만 gpt는 잘못된 cidr라고 지적함 우선 10.10으로 진행함)
+ Target : Transit Gateway → 0304-TGW-KR-01
+ 변경사항 저장
<img width="991" height="372" alt="image" src="https://github.com/user-attachments/assets/233589f8-c474-41a3-9051-6ef7183f00b5" />
<img width="725" height="444" alt="image" src="https://github.com/user-attachments/assets/983e13fc-eb42-424c-b2c6-7437e1e883bf" />
<img width="725" height="444" alt="image" src="https://github.com/user-attachments/assets/6fda5c99-2f81-4543-9fe6-1b70023d4f6b" />

(3) spoke private1 편집
+  0304-SPK-KR-01-rtb-private1-ap-northeast-2a tjsxor
+  라우팅 편집 클릭
+  라우팅 추가 클릭
+  Destination : 10.0.0.0/16
+  Target : Transit Gateway -> 0304-TGW-SPK-01
+  변경사항 저장
<img width="909" height="450" alt="image" src="https://github.com/user-attachments/assets/8963343b-2fde-4f0f-b17f-182f936b23f4" />
<img width="626" height="165" alt="image" src="https://github.com/user-attachments/assets/7fe5ae25-4683-45f4-968d-91d0c9521f38" />

<img width="914" height="638" alt="image" src="https://github.com/user-attachments/assets/8bd4080a-1286-4105-a247-b59edf63a2c5" />
<img width="731" height="452" alt="image" src="https://github.com/user-attachments/assets/d8b67a52-3743-4d1f-8972-72a4d6c7b379" />

(4) spoke private2 편집
+ 0304-SPK-KR-01-rtb-private2-ap-northeast-2b 선택
+ 라우팅 편집 클릭
+ 라우팅 추가 클릭
+ Destination : 10.0.0.0/16
+ Target : Transit Gateway -> 0304-TGW-SPK-01
<img width="913" height="377" alt="image" src="https://github.com/user-attachments/assets/134084d9-1739-456c-be9f-0e094a320707" />
<img width="626" height="165" alt="image" src="https://github.com/user-attachments/assets/ef468cfc-0658-4915-8a27-15603c891b2e" />
<img width="911" height="607" alt="image" src="https://github.com/user-attachments/assets/f4cf2170-a47f-4384-aa3b-b57ef7611547" />
<img width="725" height="443" alt="image" src="https://github.com/user-attachments/assets/464fb522-0df6-459e-814f-f27e99eac90b" />

## 6. EC2 생성
**EC2 -> 인스턴스 시작**     
(1) 설정
+ Name : 0304-SPK-NGINX-01
+ AMI : Ubuntu
+ Instance type : t3.micro 또는 t4g.micro
+ 네트워크
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

 
## 7. Bastion 서버 생성 (Hub)
**EC2 -> 인스턴스 시작**      
+ Name : 0304-HUB-BASTION-01
+ 애플리케이션 및 OSI : Ubuntu
+ 인스턴스 : t3.micro
+ 키페어 : 0227-aws-bastion-key (기존 키 페어)
+ 네트워크
  + VPC : 0304-HUB-KR-01
  + Subnet : private1 (10.0.30.0/24)
+ Public IP : Disable
+ 보안그룹생성
  + 이름 : 0304-SPK-NGINX-SG-01
  + SSH : 22
  + source = 0.0.0.0/0
 
<img width="1234" height="743" alt="image" src="https://github.com/user-attachments/assets/c648ffa3-e658-4b21-8eb7-07ece860dfd1" />
<img width="1220" height="731" alt="image" src="https://github.com/user-attachments/assets/74b401da-b14e-4efe-a0d3-466229809ef7" />
<img width="1218" height="838" alt="image" src="https://github.com/user-attachments/assets/dfe222e0-23bf-4489-a421-96377066758b" />


## 7. Bastion 접속
**EC2 -> Bastion -> 연결**     
+ EC2 Instance Connect  또는 Cloudshell 
  + 접속하면 ubuntu@ip-10-0-30-x으로 뜸

## 8. Nginx 설치 : NAT 만든 후에 
**Bastion에서**    
+ ssh ubuntu@10.10.20.x 접속
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
**Bastion에서**
```
ping 10.10.20.x
```

+ 정상 : 64 bytes from

## 9. SSH 테스트
+ ssh ubuntu@10.10.20.x
  + 접속 안되면 키 생성
    ```
    ssh -i key.pem ubuntu@10.10.20.x
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
