
   # 2026.02.12.thu #
   ## vmmaster1 : ssh connection issue ##
```
   sudo nano /etc/ssh/sshd_config
```
+ /etc/ssh/sshd_config에서 수정할 내용: 아래 두 줄 #로 주석처리
+ 확장자가 없는 리눅스 설정 파일로 SSH 서버의 설정을 담당.
```
# PasswordAuthentication yes
# KbdInteractiveAuthentication yes
```
```
# 적용
   sudo systemctl restart ssh
```
루트랑 세갈래 나누는 인그레스를 만들었다 => 실제로 백엔드 서버에서 데이터 가져와서 하게되는 것     
DB는 insert해서 이름이랑 정보 팀장님이 바꿔주실 것      
프로젝트 보고서에 프론트 엔진과 백엔드 뭐 썼는지 적어야 함
프로젝트
1)pvc나 host로 하는 것
2)nfs로 하는 것 :외부
