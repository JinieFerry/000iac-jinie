# 155. ssl Https 서버 구축 실습

# 0. Ubuntu-https (http 서버 +  snort 보안 역할) VMware에 설치

실습용 빠르게 생성하기 위해 Typical
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/7c199841-bd6e-4f2f-91b6-c9c8f9b4ed81" />

다운로드 해 둔 Ubuntu 20.02.6 (D드라이브> VMware > ISO)
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/840f6c92-2cba-49e1-8b64-93282b2da1c0" />

유저네임과 패스워드 기억 (로그인 시 사용)
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/a200a7bb-4d52-4852-8f14-ed51d51616da" />

다른 vm(공격자 kali, 취약서버 metasploitable2와 마찬가지로 D드라이브>VMWare>VM)
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/5c3a4fe0-3820-4931-a384-22778200c2ca" />

속도 빠르고 안전한 single file 권장
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/bad77d4e-7b12-4e18-8cd5-fe2637d59d10" />

# 00. 실습 준비

`ip a` 아이피 확인
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/14e0c56c-c734-4625-86b0-d8518188de78" />

'ping google.com' 인터넷 체크 (crtl + c 로 종료)
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/fff26662-e099-4b05-98bb-b3b6e0b068ec" />

업데이트
```
sudo apt update
sudo apt upgrade -y
```
