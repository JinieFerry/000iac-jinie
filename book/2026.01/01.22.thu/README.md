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
        - 192.168.10.187/16
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```
<img width="946" height="1040" alt="image" src="https://github.com/user-attachments/assets/4ec790e0-1b48-413c-8a43-eec874b9b920" />

3) 저장 후 적용

3-1. vi에서:

ESC
:wq


3-2/ 그 다음 적용:
```
sudo netplan apply
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
master@vmmaster15:~$ ping -c 3 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=30.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=28.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=28.5 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2444ms
rtt min/avg/max/mdev = 28.514/29.262/30.616/0.959 ms
```
정상 기준:

enp0s3 → 192.168.10.187/16

default via → 192.168.10.1

ping 성공
