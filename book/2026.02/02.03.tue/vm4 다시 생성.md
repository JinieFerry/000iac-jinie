===
Jinie Log
===

#### 미해결 이슈: vm4 로그인 패스워드 입력 후 무응답 
<img width="1117" height="1031" alt="image" src="https://github.com/user-attachments/assets/26ab7ac9-1cdc-4e4f-87c9-88585769d3b8" />

+ D드라이브에 vm3복제해서 vm4 생성 시도 -> 로그인 패스워드 입력 후 무응답   
+ D드라이브에 vm3복제해서 vm5 생성 시도 -> 로그인 패스워드 입력 후 무응답   
+ C드라이브에 vm3복제해서 vm4 생성 시도 -> 로그인 패스워드 입력 후 무응답 => ctrl+art+fn+fn3로 터미널 모드로 진입 성공

#### 0. vm4 셋팅: 고정 아이피, 게이트 웨이 설정
<img width="847" height="416" alt="image" src="https://github.com/user-attachments/assets/502c4fdf-6f84-4d69-a3e9-f301b8496a61" />
```
# 아이피 확인: vm3로 나옴 (수정 전)
$ ip a
# vm4 아이피 할당: 192.168.115.4/16
# 01-netcfg.yaml인지  50-cloud-init.yaml인지 확인

$ cd /etc/netplan
$ ls # 여기에 있는 yaml 파일을 다음에서 수정

# 수정
$ sudo vi /etc/netplan/50-cloud-init.yaml
# addresses:vm4의 고정 아이피 주소 = 192.168.115.4/16 으로 수정
# via:게이트 웨이 = 192.168.10.1

# 저장하고 나가기
$ wq!

# v4 고정아이피와 게이트웨이 적용
$ sudo netplan apply

# 확인
$ ip a
# enp0s3 , 192.168.115.4/16
$ ping -c 2 8.8.8.8
```
+ Mac address 새로고침 완료
<img width="804" height="346" alt="image" src="https://github.com/user-attachments/assets/772cd277-aacf-4b2d-b17a-69c98ed0fb90" />

+ 호스트파일 수정
```
# 호스트파일 수정
$ sudo vi /etc/hosts
# 127.0.1 vmmaster1을 vm4로 수정
```

+ SSH 키 등록
1. 우선 password 선택하고 master pw입력해서 로그인
<img width="990" height="1034" alt="image" src="https://github.com/user-attachments/assets/d34a5a34-6a5c-48e1-82b2-dde33cda3f40" />
<img width="992" height="342" alt="image" src="https://github.com/user-attachments/assets/76a4d362-9126-48c2-a95f-9f381eca9c45" />

2. vm4 등록정보에서 id_rsa 선택하고, 등록정보에서 id_rsa 키 전체 복사
<img width="733" height="641" alt="image" src="https://github.com/user-attachments/assets/cb5a598f-fbd9-4e4e-950f-026446821970" />

3. vm4 서버 공개키 등록
```
#vm3 복제한거라 .ssh 디렉토리 이미 있음 
$ cd ~/.ssh
$ ls
$ vi authorized_keys

```

#### vm1 authrized_keys 파일 비어있어서 ssh키 다시 가져오기
```
$ ssh-copy-id master@vm1
# 확인
$ ssh master@vm1 # 패스워드 묻지않고 접속되면 성공
```

