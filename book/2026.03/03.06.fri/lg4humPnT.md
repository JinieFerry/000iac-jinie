# 실습 복습

## 01. VPC 생성
+  0306-net-01-vpc
<img width="1307" height="790" alt="image" src="https://github.com/user-attachments/assets/39157981-883f-469a-8677-7218d2136068" />

## 02. 서브넷 설정 편집 : IPv4 자동 주소 할당 
+ public 1
<img width="1007" height="642" alt="image" src="https://github.com/user-attachments/assets/ee28a232-cf94-4feb-a238-16a766fb1faf" />

+ public 2
<img width="994" height="603" alt="image" src="https://github.com/user-attachments/assets/b6b19c1a-390b-4a70-bf89-5d7cd24251ef" />

## 03. 보안그룹 생성 : 인바운드 규칙 추가 http -> 0.0.0.0/0
+ ALB용 : 0306-sg-alb-01
+ vpc: 0306-net-01-vpc
<img width="797" height="465" alt="image" src="https://github.com/user-attachments/assets/6d9d3ae7-b2f7-498e-b329-96da8a423c11" />
+ EC2용 : 0306-sg-ec2-01
+ vpc: 0306-net-01-vpc
<img width="977" height="697" alt="image" src="https://github.com/user-attachments/assets/4cd09719-b792-4c43-a409-6b12aa8d891c" />
<img width="1302" height="466" alt="image" src="https://github.com/user-attachments/assets/93c66a42-edbb-41a5-a80a-f71a92c684be" />

## 04. 시작 템플릿 : 0306-launch-template-01
<img width="642" height="412" alt="image" src="https://github.com/user-attachments/assets/b7dd49be-e099-41ce-85eb-39f9bf28ab11" />
<img width="641" height="680" alt="image" src="https://github.com/user-attachments/assets/c821546a-a6f7-429a-bdbb-b65fb0453887" />
<img width="642" height="745" alt="image" src="https://github.com/user-attachments/assets/a437ff23-0ec0-4fe5-974e-c4d0bba055ca" />
<img width="574" height="103" alt="image" src="https://github.com/user-attachments/assets/4bdfacbb-d131-461a-a2f4-999160e3afed" />

+ 새 IAM롤 설정 : 권한 정책 AmazonSSMManagedInstanceCore
<img width="673" height="457" alt="image" src="https://github.com/user-attachments/assets/03e3ddcd-7c94-4185-b3e2-14abe01c4970" />
<img width="1272" height="345" alt="image" src="https://github.com/user-attachments/assets/731fbd2c-8b07-4f14-b2e5-761a6fca52a3" />
<img width="1027" height="726" alt="image" src="https://github.com/user-attachments/assets/9eb13ee3-574a-4757-a2de-e0c5dba16fc6" />


