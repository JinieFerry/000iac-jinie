# 2026.03.03.tue : “네트워크 위에 실제 서비스(RDS)를 올려서 외부 서비스(LG Service)가 DB에 접속하게 만들기”

VPC → RDS 생성 → S3에 SQL 업로드 → RDS에 데이터 넣기 → 보안그룹 열기 → 접속 테스트     

## 0. aws vpc 2개 생성 : 버지니아 주, 서브넷 2개씩

### 콘솔로 생성 0303-vpc-a

(0) 0303-vpc-a 생성
<img width="985" height="905" alt="image" src="https://github.com/user-attachments/assets/06430ccb-7de7-4777-891d-01a309f51fe2" />

(1) 0303-subnet-A-1 , A-2 생성

+ 0303-subnet-A-1
<img width="1003" height="1033" alt="image" src="https://github.com/user-attachments/assets/2c9f3c64-5c3e-4213-a5fe-3c980774731f" />

+ 0303-subnet-A-2
<img width="1003" height="1033" alt="image" src="https://github.com/user-attachments/assets/569cbbe5-1c6f-4317-a687-393a5989e32e" />


(3) 0303-vpc-b 생성
<img width="1003" height="1033" alt="image" src="https://github.com/user-attachments/assets/450fa081-5941-45f7-be2b-6982109e72ab" />

(4) 0303-subnet-B-1 , B-2 생성

+ 0303-subnet-B-1
<img width="1003" height="1033" alt="image" src="https://github.com/user-attachments/assets/acfade78-6aa8-45f9-96d3-77152afef88a" />

+ 0303-subnet-B-2
<img width="1003" height="1033" alt="image" src="https://github.com/user-attachments/assets/2db5db4e-ea79-4952-a018-872c146a0cd6" />

## 1. 라우팅 테이블 확인 | rtb- 로 시작함

+ 0303-vpc-a 라우팅 테이블 : rtb-0b5710fd840ba2e00
<img width="988" height="714" alt="image" src="https://github.com/user-attachments/assets/10624727-e249-41b4-bbf4-f57319061ed1" />

+ 0303-vpc-b 라우팅 테이블 : rtb-047e89560d9036e7e
+ <img width="996" height="690" alt="image" src="https://github.com/user-attachments/assets/d0daab4f-83ad-48b3-9d8f-062ab66ebf0a" />

## 2. VPC Peering 연결 생성
(0) 콘솔 이동 : VPC → 피어링 연결 → 피어링 연결 생성     

(1) Peering 생성 : 0303-vpc-a-b-peer (0303-vpc-a와b의 양방향 peering)
### 설정값

이름: 0303-peer-a-b

요청 VPC: 0303-vpc-a

계정: 같은 계정

리전: 같은 리전 (us-east-1)

수락 VPC: 0303-vpc-b

(2) CIDR 안 겹침 확인

10.10.0.0/16

10.20.0.0/16

→ 문제 없음 => 생성 클릭

(3) Peering 상태 확인
- pending-acceptance → accept
- 상태: Active

+ 상태: 수락 대기 중 (직접 요청 수락) -> 활성
<img width="1717" height="176" alt="image" src="https://github.com/user-attachments/assets/d724a91f-de08-4f01-bacd-9c434eebd913" />

+ 직접 작업 > 요청 수락
<img width="803" height="94" alt="image" src="https://github.com/user-attachments/assets/37607e1c-3c6a-4374-8277-fce861ed5889" />
<img width="808" height="689" alt="image" src="https://github.com/user-attachments/assets/d9289c43-e5b4-4d7f-9948-31314b66548f" />

+ 활성 상태
<img width="801" height="699" alt="image" src="https://github.com/user-attachments/assets/cd5d9132-9a72-4a53-aecf-19bedda57cd5" />
<img width="1724" height="129" alt="image" src="https://github.com/user-attachments/assets/35ab7a7c-8cea-4fa4-a66f-8a2f526aec71" />

✔ Peering 상태: Active
✔ CIDR 안 겹침
✔ 같은 계정 / 같은 리전


**"초록 메시지 "라우팅 테이블에 피어링된 VPC에 대한 경로를 추가해야 합니다" =  네트워크 연결선은 깔린 상태**     
VPC A ↔ VPC B 사이에 “선은 연결되었지만 라우팅 테이블에 경로가 없어 실제 트래픽은 아직 못 감 => 라우팅 테이블 수정해야 함.

## 03. 0303-vpc-a 라우팅 테이블 수정 : a에서 b로가는 길 성공 = 아직은 단방향 a -> b
(0) 콘솔 이동 : vpc > 라우팅 테이블 > rtb-0b5710fd840ba2e00 (위에서 확인한 라우팅 테이블 중 요청자  0303-vpc-a 라우팅의 라우팅 테이블)
<img width="1006" height="479" alt="image" src="https://github.com/user-attachments/assets/b914af10-a730-4565-aaad-c8af84192e4d" />

(1) 라우팅 추가: 10.10.0.0/16만 있는 상태 -> 목적지 : 10.20.0.0/16 & 대상: pcx-046fc5c1bb85b9ea2 추가
+ 라우팅 추가: 10.20.0.0/16
<img width="982" height="260" alt="image" src="https://github.com/user-attachments/assets/da3c8d61-e4ee-4161-b43b-1281e380a0af" />

+ 대상 : 피어링 연결 선택 -> 자동으로 pcx- -> 검색: pcx-046fc5c1bb85b9ea2 선택 -> 변경 사항 저장
<img width="991" height="478" alt="image" src="https://github.com/user-attachments/assets/29e6ec62-e732-43e4-87bf-633b3f9c66f4" />
<img width="571" height="218" alt="image" src="https://github.com/user-attachments/assets/61ac986d-c863-40c4-bfc5-dc6d14268f90" />
<img width="546" height="256" alt="image" src="https://github.com/user-attachments/assets/7b1a18da-a4db-4cfa-b03c-c0548ba9bcc2" />
<img width="984" height="249" alt="image" src="https://github.com/user-attachments/assets/d1f51100-2e24-44e6-bec8-cac4b07bb0ea" />

+ 성공적으로 1개의 라우팅을 생성했습니다. : 10.20.0.0/16
<img width="793" height="470" alt="image" src="https://github.com/user-attachments/assets/0331f18a-1593-4547-87d3-af7bed75aabf" />

## 04. 0303-vpc-b 라우팅 테이블 수정 : b에서 a로 가는 길도 성공 = 이제 양방향 a <-> b

(0) 콘솔 이동: vpc > 라우팅 테이블 > rtb-047e89560d9036e7e (위에서 확인한 라우팅 테이블 중 수락자  0303-vpc-b의 라우팅 테이블)
<img width="1002" height="440" alt="image" src="https://github.com/user-attachments/assets/87f1bbd3-5ef4-4192-ba17-943faaee2f95" />

(1) 라우팅 추가: 10.20.0.0/16만 있는 상태 -> 목적지 : 10.10.0.0/16 & 대상 : pcx-046fc5c1bb85b9ea2 추가
<img width="995" height="224" alt="image" src="https://github.com/user-attachments/assets/9c032053-c8ee-4973-94fb-db5ee804ed00" />


## 후이즈 & 가비아 회원가입 : 
+ 후이즈 입력 후 회원가입 요청 -> 회원가입 완료
<img width="914" height="825" alt="image" src="https://github.com/user-attachments/assets/1f8958ab-f984-4bbf-be3d-3dc27333180c" />
<img width="886" height="887" alt="image" src="https://github.com/user-attachments/assets/96243eec-289b-4889-a067-a91c0b7762ae" />

