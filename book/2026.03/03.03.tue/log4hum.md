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
