+ VMWare 설치 : create new virtual machine -> iso 말고 intstall later로
+ **800/ETC 참고**
0) 설치 전 재부팅
1) craete a new virtual machine
2) custom
<img width="913" height="802" alt="image" src="https://github.com/user-attachments/assets/225de7a6-e37a-45de-94f4-c29c3ad6c72c" />
3) workstation 25H2 or later
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/b2c4c948-7fae-49ef-ba49-3cc1677d4212" />

4) I will install the operating ~
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/78d38283-66f6-4ee4-8b77-25d3d1f7913b" />

5) Guestion operating system : Linux , Version: Ubuntu 64-bit
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/25f72ec0-ecb2-4026-996a-4c1e9defbc01" />

6) Location은 D드라이브에 VMWare 새폴더 생성
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/71b36e3e-368c-4b3e-8fea-bd7c3d2ea6b5" />

7) 프로세서는 2 ,1 그대로
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/d900d811-6d2e-4427-9789-9a8c8e6a00b4" />

8) 메모리도 4096 그대로
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/44b460c4-23d3-4dd3-bede-33fc53ffaf63" />

9) Use ~ (NAT) 선택
<img width="428" height="430" alt="image" src="https://github.com/user-attachments/assets/cf7c0ff7-6710-49e6-ba66-97941aef6d15" />
10) LSI Logic
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/53c6991a-da87-46cb-acbe-31f95af9d110" />
11) SCSI
<img width="919" height="1049" alt="image" src="https://github.com/user-attachments/assets/eaf92c11-04bd-4bd7-a00f-1887e9e6c651" />

12) Create a new virtual disk

<img width="756" height="716" alt="image" src="https://github.com/user-attachments/assets/c95a1d2b-aa4c-49ed-b5df-e5902a76f380" />
13) Maximum distk size : 30GB & a single file
<img width="756" height="716" alt="image" src="https://github.com/user-attachments/assets/5ce8c78f-c54d-4d76-b928-cfb9f999edd0" />
14) specipic : Hardware > Processors> Virtualzation engine : 첫번째만 체크하기
15) Nat > Advanced ... > Mac Address Generate 버튼으로 생성
16) USB Controller > Connections : USB 3.2
17) 디스플레이 스트레치 모드

### VM > Settings > Options 생김
18)  > Shared Forders : Always enabled > Add : d드라이브에 vm_share
19)  > Advanced : UEFI (안되면 새로 하나 다시 만들기)
<img width="759" height="734" alt="image" src="https://github.com/user-attachments/assets/60a31e14-08bc-4ac4-bfa4-5b97596f5f58" />
<img width="759" height="734" alt="image" src="https://github.com/user-attachments/assets/6023a48b-cc81-4904-a337-86b31f738790" />
<img width="428" height="379" alt="image" src="https://github.com/user-attachments/assets/f3d735bc-ce3b-4584-ac03-a61bcbb7a458" />
<img width="759" height="734" alt="image" src="https://github.com/user-attachments/assets/ed819af5-c875-479c-bfae-60c91f2409c0" />
20) CD/DVD > Connection : USE ISO ~ : Browse... D드라이브에 다운로드 해 둔 ubuntu 20.04.6 sio 이미지 파일 선택
<img width="759" height="734" alt="image" src="https://github.com/user-attachments/assets/686c0fe8-13d3-4af2-b6c0-e0b34ab886e3" />



