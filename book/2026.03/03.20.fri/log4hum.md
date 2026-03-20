# 최종 픽스 부탁드림

## 0. 실습 구조 : 셋 다 같은 네트워크에 있어야 함 (그래서 '어댑터에 브릿지' 설정)
```
[내 PC]
   │
   ├─ Kali (공격자)
   ├─ Metasploitable2 (취약 서버)
   └─ Target2 (정상 서버)
```

# 00. 진행순서
STEP 1 

Kali OVA 설치

Bridge 설정

STEP 2

Metasploitable2 OVA 설치

Bridge 설정

STEP 3

둘 다 켜고 ping 확인

STEP 4

Target2 설치

IP 수동 설정

최종

nmap 스캔 → metasploit 공격


## 1. 설치 전략
+ OVA/VM 파일로 설치 (ISO 설치는 오류 많아서 생략)
+ ISO = 설치 중 ssh,네트워크,패키지 꼬일 가능성 농후
+ OVA = 이미 세팅 된 상태 그대로 설치

## 2. VM 구성

1) Kali (공격 서버)

타입: Linux / Debian 64bit

방식: OVA (702-14)

네트워크: Bridge

2) Metasploitable2 (취약 서버)

타입: Linux / Ubuntu 32bit

방식: OVA (702-200)

네트워크: Bridge

3) Target2 (정상 서버)

방식: ISO or 직접 설치 (702-100)

네트워크: Bridge

IP:

192.168.10.1xx (xx = 본인 PC 번호 ex. 1 + 15 = 115)

## 3. 설정 시 통일해야하는 조건 (여기서 많이 터짐)

**VirtualBox → 설정 → 네트워크**

어댑터 1: Bridge Adapter

어댑터 선택:
→ 본인 실제 인터넷 (Wi-Fi or Ethernet)

## 4. IP전략

기본 : 192.168.1+PC번호.xxx 

기본 대역으로 안 될 시 10점 대로 낮춤: 192.168.10.1+PC번호    
예시 (성의진 : PC 15번)    
Kali:           192.168.10.115   
Metasploitable: 192.168.10.116   
Target2:        192.168.10.117   
Gateway:        192.168.10.1    

## 5. Target2 네트워크 설정 (여기서 많이 막힘)

Ubuntu 기준:
```
sudo vi /etc/network/interfaces
```
+/etc/network/interfaces
```
auto eth0
iface eth0 inet static
address 192.168.10.115
netmask 255.255.255.0
gateway 192.168.10.1
dns-nameservers 8.8.8.8
```
적용:
```
sudo ifconfig eth0 down
sudo ifconfig eth0 up
```

## 6. SSH 안될 때 
```
sudo apt update
sudo apt install openssh-server -y
sudo service ssh start
```

## 7. 통신 확인 (여기까지 되면 성공)
Kali에서
```
ping 192.168.10.116
ping 192.168.10.117
```
→ 둘 다 살아야 정상
