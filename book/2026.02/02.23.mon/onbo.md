# 2026.02.23.월 이론
+ D-Dos 공격: 서비스 자체를 못 하게 하는 것, 도스 공격을 제외하고는 계정과 권한 관리에서 가장 사고가 많이 발생함
 : 임시로 할 일 들이 많기 때문에 관리자가 배포를 잊어서 사고 발생 -> 자동으로 복귀하게 되도록 시스템이 생겼음   

+ IAM: 계정과 권한 관리
  : 대부분 json정책으로 정의하며 최소 권한의 원칙 -> zero trust 원칙으로 변모: 인증된 사람도 모든 행동에 있어서 감시하는 것

 <img width="950" height="839" alt="image" src="https://github.com/user-attachments/assets/6642c1ff-1644-4e0e-a64e-25ce490e3c5b" />

+ Access key와 Secret key : pem, 서버에서는.ssh에 저장했음
  -> EC2 인스턴스, lambda 함수 등 aws 내부 컴퓨팅 자원에는 절대 액세스 키 발급하지 않고 IAM Role 부여해 메다데이터 서비스를 통해 임시 자격 증명을 사용해야 함

## 보안 사고 사례
<img width="935" height="559" alt="image" src="https://github.com/user-attachments/assets/447095a3-69aa-46df-8898-398bcd471171" />
+ 유명 금융사 해킹 사고의 핵심 메커니즘  
SSRF (Server-Side Request Forgery, 서버 측 요청 위조)
테라폼은 암호가 보이기 때문에 잘 숨겨놔야 함 : terraform state file에 암호가 적나라하게 텍스트 파일로 들어감(테라폼이 읽어야 해서)    
-> 버킷을 만들면 테라폼 외에 아무도 못 들어가게 해야 함

<img width="949" height="488" alt="image" src="https://github.com/user-attachments/assets/042a3016-d28e-4ea8-be7a-2aa6a2f048d1" />
+ Confused Deputy 혼동된 대리자 취약점
  

<img width="961" height="592" alt="image" src="https://github.com/user-attachments/assets/0a05b56d-1284-492c-a9aa-bbf089b9886a" />
+ 액티브 액티브는 프론트, 액티브 패시브는 DB에서 쓰기 좋음


<img width="939" height="377" alt="image" src="https://github.com/user-attachments/assets/89bbd29e-7f59-4124-b66c-afa084dc45b3" />
+ DNS 쿼리 날릴 때 라우트 53으로 받음 (서비스 포트)
+ 밖에서 들어 올 때 우리 아이피를 모르니까 주소와 ip를 가지고 우리 서버로 들어오도록 알려주는 것 = 우리는 DNS 필요 없음
+ 우리 회사 안에 서버가 많을 땐 DNS 필요: IP로 다 프로그래밍 하면 좋지 않아서 변수로 박아두는 것 <- DNS, 제일 쉬운 방식:hosts

+ Horizental -> Vertical 위: CPU,Memory + GPU (ex:GCT)
