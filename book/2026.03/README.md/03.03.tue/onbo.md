# 2026.03.03.tue
## DNS 실습을 위한 사전 준비 단계
1. VPC A 생성
2. VPC B 생성
3. 서로 통신 안 됨 확인
4. Peering 연결
5. 라우팅 추가
6. 이제 통신됨 확인

+ **DNS는 이름 해석 → IP 반환 → 실제 IP로 TCP 연결의 과정으로 이름만 해결한다.**
+ IP가 다른 VPC에 있는데 라우팅 없으면 연결 실패한다.
+ **따라서 실제 통신을 위해서는 라우팅이 필요하다.**
  
## putto4u의 100. Route53 DNS/0001. DNS 작동원리.html# DNS 작동 원리 정리에 대하여 재정리

DNS는 단순한 “이름 → IP 변환” 도구가 아니다.  
인터넷 전체가 동작하는 **계층적 위임 시스템(Delegation System)** 이다.

---

## 1. DNS 계층 구조

DNS는 아래와 같은 계층 구조를 가진다.

Root (.)
 └── TLD (.com, .net, .kr ...)
      └── example.com
           └── www.example.com

핵심은 두 가지 이다.

- 상위 서버는 **하위 서버의 위치만 알려준다**
- 실제 IP는 최종 권한 서버(Authoritative)가 가지고 있다

---

## 2. DNS 구성 요소

### 1) Stub Resolver
- 사용자 PC 내부 DNS 클라이언트
- /etc/hosts, 로컬 캐시 확인
- 직접 루트 서버에 가지 않음

### 2) Recursive Resolver
- ISP 또는 8.8.8.8 같은 서버
- 루트 → TLD → 권한 서버까지 대신 질의
- 결과를 캐싱

### 3) Root Server
- 전 세계 13개 논리 루트
- 실제로는 Anycast로 수천 대 운영
- IP를 주지 않고 TLD 위치만 안내

### 4) Authoritative Server
- 실제 Zone 데이터 보유
- 최종 IP 응답

---

## 3. DNS 질의 흐름 (www.example.com 접속 시)

1. 브라우저가 로컬 캐시 확인
2. 재귀 리졸버에 질의
3. 루트 서버 → .com 위치 반환
4. TLD 서버 → example.com 권한 서버 반환
5. 권한 서버 → www IP 반환
6. 재귀 리졸버 캐싱 후 클라이언트에 전달
7. 브라우저가 해당 IP로 TCP 연결

---

## 4. 주요 레코드 유형

### A
IPv4 주소 매핑

### AAAA
IPv6 주소 매핑

### CNAME
별칭 (로드밸런서, CloudFront 연결 시 자주 사용)

### MX
메일 서버 지정 (숫자 낮을수록 우선순위 높음)

### NS
권한 서버 지정 (Delegation의 핵심)

### PTR
역방향 조회 (IP → 도메인)

### TXT
SPF, DKIM, 소유권 검증 등

---

## 5. BIND 설정 핵심 포인트

### recursion no;
→ 권한 서버 전용 설정

### allow-transfer { none; };
→ 불필요한 Zone Transfer 차단

### dnssec-validation auto;
→ DNSSEC 서명 검증

---

## 6. TTL과 마이그레이션 전략

### 일반 운영 시
TTL: 1시간 ~ 24시간 권장

### 서버 이전 시

1. 이전 며칠 전에 TTL을 300초로 낮춤
2. Cut-over 시 IP 변경
3. 안정화 후 TTL 원복

주의:
이미 캐싱된 TTL은 즉시 줄어들지 않는다.

---

## 7. 보안 위협 정리

### Cache Poisoning
→ DNSSEC으로 무결성 검증

### Amplification DDoS
→ 오픈 리졸버 금지
→ Recursion 외부 차단

### DoH / DoT
→ DNS 암호화
→ 기업 환경에서는 트래픽 통제 이슈 발생 가능

---

## 정리

DNS는

- 계층적 위임 구조
- 캐싱 기반 성능 시스템
- 보안 설계 필수 인프라

Route53을 이해하려면
이 위임 구조부터 정확히 알아야 한다.
