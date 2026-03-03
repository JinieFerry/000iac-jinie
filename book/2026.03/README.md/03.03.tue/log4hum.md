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

## 05. EC2 인스턴스 생성
1. 인스턴스 시작

AMI: Ubuntu or Amazon Linux

인스턴스 유형: t3.micro

VPC: 기본 VPC 써도 됨

퍼블릭 IP 자동 할당: 켜기

보안그룹:

SSH (22) → 내 IP

HTTP (80) → 0.0.0.0/0 

+ 사용자 데이터 추가
```
#!/bin/bash
apt update -y
apt install nginx -y
systemctl start nginx
systemctl enable nginx

cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>KJJ- art Server</title>
</head>
<body>
<h1>Hello, one rock, bokdang, boksun, bokchi . I am your hyung. </h1>
<p>This is my EC2 server.</p>
</body>
</html>
EOF
```
<img width="475" height="440" alt="image" src="https://github.com/user-attachments/assets/3601c434-d3ea-4910-b7fb-f1c015e372a2" />
<img width="999" height="815" alt="image" src="https://github.com/user-attachments/assets/d3fe0bd9-d9a3-47c2-ade5-36257edbaf97" />
<img width="643" height="812" alt="image" src="https://github.com/user-attachments/assets/c8aea9a4-796e-4c71-81dd-f91f8d4f5e15" />
<img width="611" height="381" alt="image" src="https://github.com/user-attachments/assets/73b63a35-9d0d-4975-a296-7c7ef759890f" />

+ 인스턴스 시작   
<img width="1898" height="198" alt="image" src="https://github.com/user-attachments/assets/a08e02c8-7900-4472-8a73-955c3dedfb68" />

+ 보안그룹에서 인바운드 규칙에 http 80 포트 추가
인스턴스 생성에서 자동 생성된 launch-wizard-1 
<img width="711" height="292" alt="image" src="https://github.com/user-attachments/assets/be9bcac9-ce7f-4744-878e-2bd4fac77081" />

+ 인바운드 규칙 추가 : HTTP 80 Anywhere-IPv4-0.0.0.0/0
<img width="938" height="201" alt="image" src="https://github.com/user-attachments/assets/bd1dc61e-d3bf-4ba5-bb58-6a7689fcbeee" />

+ 브라우저 접속 성공 : http://100.48.90.46
<img width="918" height="240" alt="image" src="https://github.com/user-attachments/assets/61796dc1-62e3-4de0-927d-3c4d866e0360" />

### Route 53 : 호스트 존 생성
+  aws > route 53 호스트 영역 생성
<img width="965" height="520" alt="image" src="https://github.com/user-attachments/assets/52c803da-efd0-43dd-8cb4-37b5af961d54" />
+ Gabia에서 구매한 frash.com 호스팅 영역 생성
<img width="1439" height="793" alt="image" src="https://github.com/user-attachments/assets/466c5425-f210-455e-874b-e6a98334e02d" />
<img width="1443" height="616" alt="image" src="https://github.com/user-attachments/assets/c2fd8ad6-e4dc-40f4-ae40-0991df087efc" />

### Gabia :  생성한 호스트 영역 타사네임서버 사용으로 추가
+ 1년으로 신청
<img width="795" height="421" alt="image" src="https://github.com/user-attachments/assets/fceedcc1-bc78-4d8e-9e4d-4d3f9c7bd16b" />
<img width="1092" height="940" alt="image" src="https://github.com/user-attachments/assets/ca08368c-163d-4fd4-887f-09bb90e121fc" />

+ 타사 네임 서버 사용 : route 53에서 자동으로 생성한 호스트영역 4개의 서버 마지막의 . 제거하고 붙여넣기     
<img width="744" height="116" alt="image" src="https://github.com/user-attachments/assets/29ffbc14-a704-4b31-9f61-50efbea1fbfd" />
<img width="881" height="831" alt="image" src="https://github.com/user-attachments/assets/f03fdcbc-e88b-461a-8db3-1c9c43b3a31c" />

+ 신청 내역 출력 (선택)
<img width="696" height="660" alt="image" src="https://github.com/user-attachments/assets/f0305687-398f-4b64-a11a-e7a249c00fe5" />

+ A 레코드 생성 
<img width="991" height="582" alt="image" src="https://github.com/user-attachments/assets/4c81a68b-1de6-498c-bf72-6438a6cb8b4f" />
<img width="995" height="685" alt="image" src="https://github.com/user-attachments/assets/66ff841e-1227-4330-8d9f-9b830fd72fa6" />
<img width="1911" height="647" alt="image" src="https://github.com/user-attachments/assets/dc6ee84d-2a6f-4131-9f5f-9e591794d217" />

## 브라우저 접속 성공
<img width="960" height="1034" alt="image" src="https://github.com/user-attachments/assets/425ec098-188e-46cc-b9b1-fc83f0153bdb" />

### 고양이 보기
+ ssh 임시로 0.0.0.0/0으로 모든 접속 허용하기
<img width="960" height="240" alt="image" src="https://github.com/user-attachments/assets/bcf4391d-77bf-4f53-843b-3bc20b6c7da5" />

+ 고양이 코드로 수정: https://us-east-1.console.aws.amazon.com/ec2-instance-connect/ssh/home?region=us-east-1&connType=standard&instanceId=i-0ab2c502aae605cd8&osUser=ubuntu&sshPort=22&addressFamily=ipv4

aws 보안그룹에서 직접 ssh 연결해서 파일 수정 
```
sudo nano /var/www/html/index.html
```

```
<!DOCTYPE html>
<html>
<head>
<title>KJJ - Art Server</title>
<style>
body {
    background-color: #ffeef5;
    text-align: center;
    font-family: monospace;
}
pre {
    font-size: 20px;
    line-height: 1.2;
}
h1 {
    color: hotpink;
}
</style>
</head>
<body>

<pre>

      /\_/\              /\_/\
     ( o   o )          ( o   o )
     (   =^=   )        (   =^=   )
      (           )      (           )
       (  (  )   (  )    (  (  )   (  )
       (__(__)___(__)_)   (__(__)___(__)_)

</pre>

<h1>Hello one rock, bokdang, boksun, bokchi ! /h1>
<p>I am your hyung.</p>

</body>
</html>
```
<img width="1004" height="1033" alt="image" src="https://github.com/user-attachments/assets/50c7fcaa-5c4c-4aab-90d7-52af21afaf27" />


## 과금 방지 : 삭제하기
(1) EC2 > 인스턴스 : 인스턴스 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/ec152f8f-81a2-4e7c-bead-d336821ea009" />

(2) 탄력적 IP: 있으면 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/c5dd7e84-4ff0-485f-8617-581368bf5886" />

(3) 볼륨 : 있으면 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/39ae0b5c-4d1b-46ca-a678-c900cf63874d" />

(4) 보안그룹 : 과금 없음 삭제는 선택사항
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/e6dd9abf-5acb-4f1d-9c4c-d3a2057e49e9" />

(5) aws > route53> 호스팅 영역 : A 레코드 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/a9d51ba6-6f86-4c7c-915b-7a4ace921bc0" />

(6) 호스팅영역 삭제
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/902d6b0a-93ea-4cf0-9b4a-748e5558ebe5" />

A 레코드 먼저 삭제해야 호스팅 영역 삭제 됨
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/2c9f7c49-04b4-4a79-9ab5-6be34710a0f9" />

## 최종 과금 확인
+ EC2 없음
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/6cca2b9a-291e-4836-aab3-e20c1bf25961" />

+ 호스트 영역 없음
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/619630e4-313b-4a57-a03c-7cd0decfad27" />

+ 탄력적 IP (Elastic IP) 없음
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/82397bc3-3b18-44a4-af74-c9d72987d3fd" />

