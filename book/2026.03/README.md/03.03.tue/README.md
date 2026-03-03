# 오후 실습
1. EC2 생성 + vpc에 소속 (Public) + Github page index.html 불러오기 - curl로 받아서 copy: -o index.html 스크립트 유저데이터에 생성
2. VPC 생성, 서브넷 생성
3. 보안그룹(디폴트 보안그룹 수정하거나 새로 생성) HTTP + SSH - us VPC  생성 , VPC 선택
4. R53에서 호스트 존 생성 : ns 영역으로 4개 생성 됨 <- 이 네개 싹 긁어서 단톡방에 업로드 -> 가비아 ns 설정
5. www 레고드 추가 EC2 pub IP 추가
6. 웹 브라우저 www.ferry.ai.kr 쳐서 자기 홈페이지 나오면 성공

 자기가 2천원으로 구매해서 해도 되고, putto의 서버 사용해도 됨 => 개인 서버 www.frash.ai.kr 만들기

 ## 실습 구조
 ```
도메인 (가비아/후이즈)
        ↓
Route53 Hosted Zone
        ↓
A 레코드 → EC2 Public IP
        ↓
EC2 (index.html)
        ↓
www.개인도메인
```
