# Kali 설정
```
# GUI 로그인 창 초기 자격 증명
Username: kalimaster
Password: 1234

# 터미널 실행 후 실무를 위한 Root 권한 획득
kali@kali:~$ sudo su -
[sudo] password for kali: # kali 입력
root@kali:~# # 루트 프롬프트 활성화 완료

# 저장소 목록 업데이트 및 설치된 패키지 업그레이드 (-y 옵션으로 프롬프트 자동 수락)
root@kali:~# apt update && apt full-upgrade -y

# 불필요한 패키지 자동 정리
root@kali:~# apt autoremove -y
```

## DNS 수정
<img width="1282" height="875" alt="image" src="https://github.com/user-attachments/assets/17593497-cd60-48fc-9e26-d1507ad575a4" />

## 한글폰트 설치

```
# passwd kalimaster #npwd : 1234

# sync
# exec /sbin/init

$ sudo apt update
$ sudo apt install -y fonts-nanum
$ sudo reboot
```


# TGserver 만들기

## 1. 게이트웨이설정
```
sudo nano /etc/network/interfaces
```
```
# 로컬 루프백 인터페이스 (기존 내용 유지)
auto lo
iface lo inet loopback

# 고정 IP 설정 (eth0 기준)
auto eth0
iface eth0 inet static
    address 192.168.10.115      # 할당할 고정 IP 주소
    netmask 255.255.255.0       # 서브넷 마스크 (Subnet Mask)
    gateway 192.168.10.1         # 게이트웨이 (Gateway) 주소
    dns-nameservers 8.8.8.8 8.8.4.4  # DNS 서버 주소 (Google DNS 예시)
```
<img width="802" height="675" alt="image" src="https://github.com/user-attachments/assets/22976606-9b28-43dd-aa1f-ef11570e0d87" />

## 2. dns 서버 설정
```
sudo nano /etc/resolv.conf
```

### /etc/resolv.conf

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

## 3.ssh 서버 설치 및 시작
```
sudo apt-get install openssh-server
service ssh status
sudo service ssh start
# sudo service ssh restart
sudo update-rc.d ssh defaults  # 부팅시 자동 서비스
```

## 4. Xshell 연결
<img width="734" height="649" alt="image" src="https://github.com/user-attachments/assets/b645315f-eea0-44a7-982d-5184b12796b0" />
<img width="734" height="649" alt="image" src="https://github.com/user-attachments/assets/7dc7c15c-f409-4aae-94c3-825ec7700b10" />

## vm TGserver에서 방화벽
```
sudo ufw disatble
```

## 저장소 초기화
`sudo nano /etc/apt/source.list`

### /etc/apt/source.list

```
deb http://old-releases.ubuntu.com/ubuntu trusty main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu trusty-updates main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu trusty-security main restricted universe multiverse
```

## apache2 설치
```
# 설치
sudo apt-get update

# 아파치2  다운로드
wget https://archive.apache.org/dist/httpd/httpd-2.2.8.tar.bz2

# 압축 풀기
tar -xvjf httpd-2.2.8.tar.bz2

# 이동
cd httpd-2.2.8
```

## TGserver 재설치

<img width="771" height="547" alt="image" src="https://github.com/user-attachments/assets/1034d922-bfdb-49bf-b792-bf8b6e312ad2" />
<img width="771" height="547" alt="image" src="https://github.com/user-attachments/assets/70f62108-2816-4cd9-8154-fac6e9c76d22" />
<img width="771" height="547" alt="image" src="https://github.com/user-attachments/assets/7d384599-d29c-47f3-9168-dbdaeb1ce200" />
<img width="771" height="547" alt="image" src="https://github.com/user-attachments/assets/3256dd2c-7314-4044-ab33-183d24b44216" />

## 설정 변경
<img width="809" height="514" alt="image" src="https://github.com/user-attachments/assets/2866e8fd-d7cd-4e8d-8fc5-c8b6e47eadbc" />
<img width="809" height="514" alt="image" src="https://github.com/user-attachments/assets/fe70533e-5157-4c6c-85d1-4b0ca4469539" />

# 시작
<img width="1067" height="884" alt="image" src="https://github.com/user-attachments/assets/ec817e8f-4c95-43f4-a8fe-bcc8a630174b" />
<img width="1026" height="843" alt="image" src="https://github.com/user-attachments/assets/b2f2bf78-c6ce-4ca2-9cf7-c5de0e290b07" />
<img width="1026" height="843" alt="image" src="https://github.com/user-attachments/assets/f686aac4-96ef-4c76-8c2c-027ce8a05970" />




