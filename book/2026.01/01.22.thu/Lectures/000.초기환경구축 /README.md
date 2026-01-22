# PC15 JinieFerry Log 2026.01.22.thu
---
## 010.고정IP네트워크설정Jinie.ver
1) 인터페이스 이름 / 현재 IP / 게이트웨이 확인  
1-1. 인터페이스 이름 확인
```
master@vmmaster15:~$ ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
enp0s3           UP             08:00:27:0f:34:47 <BROADCAST,MULTICAST,UP,LOWER_UP>
```
1-2. 현재 IP 확인
```
master@vmmaster15:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp0s3           UP             192.168.10.187/16 metric 100 fe80::a00:27ff:fe0f:3447/64
```
1-3. 게이트웨이(기본 라우트) 확인
```
master@vmmaster15:~$ ip route | grep default
default via 192.168.10.1 dev enp0s3 proto dhcp src 192.168.10.187 metric 100
```

2) Netplan 파일 확인 후 편집  
2-1. 파일 목록
```
ls /etc/netplan/
```

2-2. 편집할 파일 선택 
둘 중에 하나 01-netcfg.yaml, 50-cloud-init.yaml
```
sudo vi /etc/netplan/50-cloud-init.yaml
```
수정 전 그대로 열면 기본 상태
```
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

수정 후
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.115.251/16 #192.168.1+자기PC번호(15)=115
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses:
          - 168.126.63.1
          - 8.8.8.8
```


3) 저장 후 적용

3-1. vi에서:

ESC
:wq


3-2/ 그 다음 적용:
```
master@vmmaster15:~$ sudo netplan apply

Socket error Event: 32 Error: 10053.
Connection closing...Socket close.

Connection closed by foreign host.

Disconnected from remote host(EC2-2) at 10:49:42.

Type `help' to learn how to use Xshell prompt.

```

4.적용확인
```
master@vmmaster15:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp0s3           UP             192.168.10.187/16 fe80::a00:27ff:fe0f:3447/64
```
```
master@vmmaster15:~$ ip route | grep default
default via 192.168.10.1 dev enp0s3 proto static
```
```
# IP 주소 확인
ip addr show enp0s3

# 게이트웨이 설정 확인
ip route

# 외부 네트워크 연결 테스트 (DNS 작동 확인)
ping -c 3 google.com
```
<img width="947" height="512" alt="image" src="https://github.com/user-attachments/assets/ca57c438-e428-447b-a157-7ab08d721100" />


5.pc서버도 바꿔주기
<img width="734" height="649" alt="image" src="https://github.com/user-attachments/assets/05a3ba67-2d60-4755-88ba-da9dd15e4158" />

=>연결 성공!
<img width="946" height="1040" alt="image" src="https://github.com/user-attachments/assets/24d0cfa9-ef81-4d71-b024-5ee835e0a0d5" />
=>호스트키 수락 및 저장
<img width="450" height="486" alt="image" src="https://github.com/user-attachments/assets/57ba31fd-5dde-4868-8ffd-5d2d64b7ee53" />


pc제어판에서 네트워크 설정 (버추얼박스)그대로 맞춰주기
<img width="809" height="514" alt="image" src="https://github.com/user-attachments/assets/f2cdeb81-8133-4f30-8cc0-baa2caf7ee6c" />

<img width="569" height="401" alt="image" src="https://github.com/user-attachments/assets/c458d0d3-3bc4-46b8-9db0-38da41f1b052" />

<img width="421" height="533" alt="image" src="https://github.com/user-attachments/assets/ae91ae20-b657-43c2-8be2-b4828129a6d8" />

<img width="465" height="518" alt="image" src="https://github.com/user-attachments/assets/c86851e8-11dd-4cd4-8355-e8455b93eef4" />

6. VM에직접 열기

```
master01 login: master
Password: (1234)
```

<img width="1080" height="875" alt="image" src="https://github.com/user-attachments/assets/19d6d384-4bcf-4f9d-b72d-928ead3302c3" />
---

 
## 000.기초작업단계 Jinie.ver
기존 vmmaster015의 설정을 새로운 설정에 맞춰 바꿔주고, 이름도 vmmaster01로 저장 
<img width="962" height="594" alt="image" src="https://github.com/user-attachments/assets/2caafbcf-f974-46fa-ab08-a5ec0bf5ce0d" />

vmmaster01을 복제해서 vm1,vm2,vm3 생성
<img width="569" height="401" alt="image" src="https://github.com/user-attachments/assets/0d43529b-10b9-4c2e-871e-ca140ab42836" />
"모든 ~ 새Mac주소 생성"체크  
vm1
<img width="962" height="594" alt="image" src="https://github.com/user-attachments/assets/813bc860-669a-4f5d-a38d-9ea42800d009" />
vm2
<img width="569" height="401" alt="image" src="https://github.com/user-attachments/assets/f682a7dc-cd6a-4268-8473-4daae0173000" />
vm3
<img width="569" height="401" alt="image" src="https://github.com/user-attachments/assets/b8a6faa0-6834-4a5a-b44a-f2a79f2af65a" />







