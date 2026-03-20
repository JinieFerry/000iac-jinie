# 최종 구조 픽스 부탁드림

# 0. 실습 구조 : 셋 다 같은 네트워크에 있어야 함 (그래서 '어댑터에 브릿지' 설정)
```
[내 PC]
   │
   ├─ Kali (공격자)
   ├─ Metasploitable2 (취약 서버)
   └─ Target2 (정상 서버)
```

# 00. 진행순서
```
1) Kali OVA 설치
    │
Bridge 설정
    │
2) Metasploitable2 OVA 설치
    │
Bridge 설정
    │
3) 둘 다 켜고 ping 확인
    │
4) Target2 설치
    │
IP 수동 설정
    │
최종) nmap 스캔 → metasploit 공격
```

# 1. 설치 전략
+ OVA/VM 파일로 설치 (ISO 설치는 오류 많아서 생략)
+ ISO = 설치 중 ssh,네트워크,패키지 꼬일 가능성 농후
+ OVA = 이미 세팅 된 상태 그대로 설치

# 2. VM 구성

1) Kali (공격 서버)

타입: Linux / Debian 64bit    
방식: OVA (702-14)      
네트워크: Bridge      

2) Metasploitable2 (취약 서버)     
타입: Linux / Ubuntu 32bit      
방식: OVA (702-200)      
네트워크: Bridge     

## 3) Target2 (정상 서버)   
방식: ISO or 직접 설치 (702-100)   
네트워크: Bridge    
IP: 192.168.10.1xx (xx = 본인 PC 번호 ex. 1 + 15 = 115)      

# 3. 설정 시 통일해야하는 조건 (여기서 많이 터짐)

**VirtualBox → 설정 → 네트워크**

어댑터 1: Bridge Adapter      

어댑터 선택: → 본인 실제 인터넷 (Wi-Fi or Ethernet)

# 4. IP전략

기본 : 192.168.1+PC번호.xxx 

기본 대역으로 안 될 시 10점 대로 낮춤: 192.168.10.1+PC번호    
예시 (성의진 : PC 15번)    
Kali:           192.168.10.115   
Metasploitable: 192.168.10.116   
Target2:        192.168.10.117   
Gateway:        192.168.10.1    

# 5. Target2 네트워크 설정 (여기서 많이 막힘)

## Ubuntu 기준:
```
sudo vi /etc/network/interfaces
```
## +/etc/network/interfaces
```
auto eth0
iface eth0 inet static
address 192.168.10.115
netmask 255.255.255.0
gateway 192.168.10.1
dns-nameservers 8.8.8.8
```
## 적용:
```
sudo ifconfig eth0 down
sudo ifconfig eth0 up
```

# 6. SSH 안될 때 
```
sudo apt update
sudo apt install openssh-server -y
sudo service ssh start
```

# 7. 통신 확인 (여기까지 되면 성공)
## Kali에서
```
ping 192.168.10.116
ping 192.168.10.117
```
→ 둘 다 살아야 정상

# 실습 진행 로그

# 1. 공격서버 설치 (Kali Linux)  : https://www.kali.org/get-kali/#kali-installer-images
kali-linux-2025.x-installer-amd64.iso 다운로드   

# 2. 타겟서버 1  = 취약서버 1 설치 (Metasploitable2) : https://sourceforge.net/projects/metasploitable/
metasploitable-linux-2.0.0 다운로드    

# 3. 타겟서버 2 = 취약서버 2 설치 (Ubuntu 14.4) : https://releases.ubuntu.com/14.04/   
ubuntu-14.04.6-desktop-amd64.iso 다운로드 (server 아님)

## 4. 폴더 정리

1) D드라이브에 VM과 ISO 폴더 생성   
2) ISO 폴더 : kali & ununtu iso 파일만 두기
3) VM 폴더 : VM폴더 안에 metasploitable2폴더 생성 후, 그 안에 meta2 zip 압축 풀어두기
<img width="946" height="113" alt="image" src="https://github.com/user-attachments/assets/9d506197-4b48-4e39-bca1-9875e860dfca" />
<img width="943" height="246" alt="image" src="https://github.com/user-attachments/assets/74bb7415-387e-4367-adc1-345dd161833e" />

# 5. Virtual Box에 타겟 서버 1 Meatasploitable2 추가 : meta

## 1) Virtual Box 실행
<img width="962" height="689" alt="image" src="https://github.com/user-attachments/assets/15760d69-f9bf-49a3-bce0-2f946dc4fde1" />

## 2-1) 새로만들기 -> metasploitable2 생성 : Linux & Ubuntu 32-bit
<img width="788" height="383" alt="image" src="https://github.com/user-attachments/assets/e77ec0e2-11a6-4ffb-bf08-a2a869ae8d69" />
<img width="788" height="383" alt="image" src="https://github.com/user-attachments/assets/ee23aa85-796f-43b3-b833-e919cbeb2bac" />
<img width="788" height="383" alt="image" src="https://github.com/user-attachments/assets/7b7f6db1-9379-4461-95f2-f97e74c31e12" />

## 3-1) 설정 수정하기 1 : 저장소
**met -> 설정 -> 저장소**
기존 SATA 디스크 삭제하고 .vmdk 추가하기   

### 삭제전 : meta.vid선택하고 삭제하기
<img width="809" height="514" alt="image" src="https://github.com/user-attachments/assets/d3eafa9d-742d-4c00-8d5f-558dbbd5496f" />

### 디스크 추가하기 : + 아이콘 > 추가 (A) 클릭 > 압축 해제해 둔 Metasploitable.vmdk 선택하기 (!! .vdi 파일 아님 !!)
<img width="962" height="812" alt="image" src="https://github.com/user-attachments/assets/f77c7390-ad68-4ac4-8a80-dd23c1a8ebda" />
<img width="946" height="533" alt="image" src="https://github.com/user-attachments/assets/7fc90f96-0ab6-4c78-9468-3db8bce9340d" />
### Metasploitable.vmdk 추가 후 선택 > 선택 > 확인
<img width="962" height="812" alt="image" src="https://github.com/user-attachments/assets/174d5f40-63a1-439b-a72a-97708304f885" />
<img width="809" height="514" alt="image" src="https://github.com/user-attachments/assets/b123bd25-98ff-42b2-a900-572db3b86a1f" />

## 3-2) 설정 수정하기 2 : 네트워크
### 어댑터에 브리지로 변경
<img width="809" height="514" alt="image" src="https://github.com/user-attachments/assets/83f51ced-ee49-4b87-a624-e7d29ee50bdf" />

## 3-3) 설정 수정하기 3 : 일반 > Features 모두 양방향
<img width="809" height="514" alt="image" src="https://github.com/user-attachments/assets/c0672332-d42b-4cf5-ba03-f9735d1b4c1c" />


## 4) 실행 & DHCP로 자동 할당된 IP 확인 
### 최초 로그인은 기본 아이디 비번으로 **Login with msfadmin/msfadmin**
<img width="738" height="490" alt="image" src="https://github.com/user-attachments/assets/5cfd1c48-21f6-4488-a36b-29f80295e159" />

### DHCP로 자동 할당된 IP  : eth0 192.168.10.3
```
ifconfig
```
<img width="738" height="490" alt="image" src="https://github.com/user-attachments/assets/4b355b98-13c6-404b-97d3-f38da740d8c7" />

## 5) meta 스냅샷 남기기 : 타겟서버1 Metasploitable2 설치 성공
<img width="962" height="689" alt="image" src="https://github.com/user-attachments/assets/8a6fd4c4-feb2-4336-a56a-16b94d81337b" />

# 6. Virtual Box에 공격자 Kali 설치

## 1) Virtual Box 실행
<img width="962" height="689" alt="image" src="https://github.com/user-attachments/assets/8916d668-4f5e-44be-90b2-15d13baf259d" />

## 2-1) 새로만들기 -> kali 생성 : 

### Linux & Debian 64-bit 선택
<img width="771" height="547" alt="image" src="https://github.com/user-attachments/assets/9eb9650d-8ac6-443c-b094-cf9ddcafcf7d" />

###  4096 MB , CPU 2 개 , Use EFI 체크하지 않음
<img width="771" height="547" alt="image" src="https://github.com/user-attachments/assets/9a75067c-08b0-46de-8cc0-4ff8ae4fe490" />

### 40.00GB로 올리기
<img width="771" height="547" alt="image" src="https://github.com/user-attachments/assets/a10ae3f2-4af3-49ac-b3f8-a62bb07ab873" />

### 완료
<img width="962" height="689" alt="image" src="https://github.com/user-attachments/assets/0a3f8da1-e115-497d-be45-124ad03bd12e" />

## 6) 실행 후 설치하기

### Graphic Install
<img width="663" height="560" alt="image" src="https://github.com/user-attachments/assets/266821f7-2394-4749-9871-025c052a1ffc" />

### 언어선택 (영어 권장) : English - United States - American English
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/d29acbcd-c860-4bc8-8d3b-d07237e74048" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/07f5bbfc-7a38-42a6-b6de-e3c9bc97878c" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/f1c3f43f-eadb-4012-a96d-77ed1a70ec7b" />

### 호스트네임 설정 (kali ~~ 권장) : kali-attacker , 도메인 네임 비워둠
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/73dbb0da-1676-4bb3-b08a-7ab688d67e91" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/0ec87151-b56b-4540-84d9-f6fd6dd6761f" />

### 유저네임 : kali-attacker (표시용)
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/3db68042-b16c-45c3-8fed-30711cf5a9de" />

### username & pw : kali(로그인용)  
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/1b9e7820-6ef9-4306-820e-61a398025cd7" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/a851e92c-8b8a-4fd0-be9d-34684fcf0bf2" />

### continue만 누르기
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/a956230b-584c-4b23-b3e7-62985751f933" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/ac25779a-9209-4a6c-817f-b53699c57872" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/bc2f3c4f-bb21-4e89-aa67-400ff5da6018" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/596020ef-0f29-4002-af4b-96103d57c970" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/e2e3fdc8-dea9-4546-a4fa-64e85ed6b807" />
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/3e0491eb-40b1-4a1e-8e2d-ae32004c9ed4" />

### yes : 설치 시작
<img width="830" height="664" alt="image" src="https://github.com/user-attachments/assets/25184cc5-953a-473e-95c7-3ac9e8f4d124" />

# 7. Virtual Box에서 타겟 서버 2 Ubuntu 14.4 설치

1) 새로생성하기 -> ubuntu14
<img width="962" height="689" alt="image" src="https://github.com/user-attachments/assets/52ca64b7-2a3d-4559-ad63-76e3c90812c2" />

3) D드라이브 , iso 선택
<img width="771" height="547" alt="image" src="https://github.com/user-attachments/assets/5a9597c9-0e60-493a-b21b-5e951a07e381" />
