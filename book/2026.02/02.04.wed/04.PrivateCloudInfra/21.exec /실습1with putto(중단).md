실제로는 28-1부터 함
010-020:yaml로 한번에 배포


#### 명령어로 한번 하고, IaC로 따로 한번 더 할 것

<img width="564" height="399" alt="image" src="https://github.com/user-attachments/assets/ead59166-6ffe-42d1-9157-795a89bec320" />
### 011.NFS IaC로 구축
0. 새 nfs 서버 생성(워커노드 복제해서 생성) Network File system
1. vm1-> nfs , vm2-> nfs_50
+ D드라이브에 생성 (vm:새폴더 생성)
+ ~ 모든 새 Mac 주소 ~

### NFS 서버 패키지 설치 및 실행:NFS는 소유권자가 중요함
```
#vmmaster1 xshell

# 0. 패스워드 입력 생략하기
$ sudo visudo
# 마지막 줄에 추가
username ALL=(ALL) NOPASSWD: ALL

# tail로도 가능
$ sudo tail /etc/sudoers

# 01 Linux 배포판에 따라 패키지 관리 도구가 다르므로 환경에 맞춰 설치합니다.
#Ubuntu/Debian:
$ sudo apt update
$ sudo apt install nfs-kernel-server #-y


# 소유권자가 중요하므로 확인
$ sudo cat /etc/passwd
# nginx 상태 확인
$ systemctl status nginx
#이동
$ cd /etc/nginx
$ cd /var
$ ls
$ cd www
$ ls -al
$ sudo vi conf.d
$ cd vi sites-avialable/

#환경파일
$ cd /etc
$ sudo vi exports
# 가장 아랫줄에 추가: 디렉토리한 정해서 호스트 네임주고 rw,싱크 줌
/srv/nfs_share      192.168.115.51(rw,sync,no_subtree_check)    
# esc: wq! 저장하고 나가기
```
<img width="884" height="309" alt="image" src="https://github.com/user-attachments/assets/59a939f0-85ff-46e9-acd6-99649687c384" />

```
# enable 생략 (트러블슈팅 실습)

# 5.클라이언트연결테스트:nginx 에서

# vm1: nginx 확인
$ sudo systemctl status nginx
$ sudo vi su
# 클라이언트 패키치 설치 (생략)
# sudo apt install nfs-common

# 마운트 실행
$ sudo mkdir -p /mnt/my_data
$ sudo mount
```
 
