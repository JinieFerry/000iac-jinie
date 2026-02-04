실제로는 28-1부터 함
010-020:yaml로 한번에 배포

#### 명령어로 한번 하고, IaC로 따로 한번 더 할 것

<img width="564" height="399" alt="image" src="https://github.com/user-attachments/assets/ead59166-6ffe-42d1-9157-795a89bec320" />
### 011.NFS IaC로 구축
0. 새 nfs 서버 생성(워커노드 복제해서 생성) Network File system
1. vm1-> nfs_50 , vm2-> nfs 생성
+ D드라이브에 생성 (vm:새폴더 생성)
+ ~ 모든 새 Mac 주소 ~

#### nfs 서버 생성(vm2복제본) 
0. 수정전 확인
```
#  수정전 확인 : 초기상태 : 모두 vm2의 셋팅으로 나오는 게 정상
$ hostname
# 아이피와 인터페이스 확인: fe80
$ ip a 
```

1. nfs서버로 셋팅 시작
```
# hostname변경
sudo hostnamectl set-hostname nfs # master pw:12345678
# 확인
hostname

# 쉘 프롬프트 네임 nfs적용
$ 
# 호스트네임 들어간 파일들 모두 nfs로 맞추기
# 확인
$ cat /etc/hostnmae
$ sudo vi /etc/hosts
# 아이피 고정 

```
