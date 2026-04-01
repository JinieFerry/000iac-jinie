# 우분투 20에 snort 2.9 설치

## snort 2.9 설치
```
# 2026.04.01.wed #
# install snort 2.9
sudo apt update
sudo apt upgrade -y
# install web server : target
sudo apt install apache2 -y
# check
systemctl status apache2
# run apache2
sudo systemctl start apache2
# auto run set
sudo systemctl enable apache2
# check
systemctl status apache2
#check the port
sudo ss -tlnp | grep 80
# restart the apache2
sudo stystemctl restart apache2
sudo systemctl restart apache2
#check
curl http://localhost
# check ip 
ip a
history | c
```

1) ip 확인해서 브라우저 접속 테스트 : 아파치2 기본 사이트 나오면 성공 : http://172.16.184.131/
<img width="890" height="1015" alt="image" src="https://github.com/user-attachments/assets/5f4f495d-0780-4974-87bf-4bfa350b53b1" />

## FTP 설치
```
# install FTP
sudo apt install vsftpd -y

# run FTP
sudo systemctl start vsftpd
sudo systemctl enable vsftpd

# check: should be 'active'
systemctl status vsftpd

# check the port : should be 21
sudo ss -tlnp | grep 21
```
## 인터넷 연결
**Edit -> Virtual Network Editor**
<img width="603" height="529" alt="image" src="https://github.com/user-attachments/assets/69e906a8-6125-4dab-aab4-c8b8920b4af0" />

VMnet8 선택 -> Restore Defaults -> Apply -> Ok
<img width="603" height="529" alt="image" src="https://github.com/user-attachments/assets/083e654b-801e-493a-bced-3e21399f4d5a" />

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        
