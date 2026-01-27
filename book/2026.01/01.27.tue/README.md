
•‘이미지’—실행 —> ‘컨테이너’  
•컨테이너 퍼머넌트 스토리지 필요 (삭제 시 데이터도 삭제)  
•개발 시 사용하는 라이브러리  중 필요한 것 만 빌드—테스트—이미지—런—>이미지:바이너리로 작동  
•유저는 상관 x, 운영 측에서는 리눅스에서 생성한 이미지를 윈도우에서 런 할 수 없음  
: 이미지를 만든 os하고 커뮤니케이션이 안되면 통신 장비가 쓸 수 없음  
•04./01./12. GitHub.com/putto4u  
•레지스트리 키(도커에서 사용) <-> GPG 키(보안 위해 사용:아무나 못 올림)  
: ‘키를 만드는 주체’
k8s에서 받은 키는 중간에 해커가 가로챌 수 없게 정식 인증 비교를 함 (keyrings)   
•도커 클라이언트는 유저 아님 cli 명령어 치는 사람을 의미함  
•소프트웨어 안에서는 ipc,같은 프로그램 안에서도 프로세서가 다르면 api : 프로세서 안에서 다른 프로세서를 불러내는 것  
•build: 이미지 생성,ps:컨테이너 보기,exec,compose  
•도커파일을 이용해서 만드는 것 오늘 실습 할 것  
•도커 file(명시적 IaC:하나의 파일)<->도커 compose(프론트앱&백엔드앱, DB&웹서버 처럼 여러개 묶고 싶을 때)  
•웹서버,도커는 5년,10년 로그 기록해야 함(금융권은 10-20년):작은 서버로 못 버텨서 로그 작업만 하는 프로그램과 서버<- 이 이미지를 합칠 때 도커 컴포즈(권장 사항은 아님)  
• 로그 이미지 묶는 것 정도로만 사용하지만 도커 compose 권장 하지 않음   <- 묶었는데 서버가 죽어버리면 에러메시지, 서비스 다운 : 격리해서 보호하기 위해서 자르는 것( 결제가 안되더라도 구매 사이트는 독립적으로 서비스 지속 권장)
=쿠보네티스가 충분히 잘 하고 있기 때문에  Docker는 생각 보다 일찍 없어 질 것 같음   (기존 사용 중 이미지 남기는
것 외에 사용 거의 없음. 신규로 사용은 쿠보네티스로 충분히 가능), 기능이 잡다함
- container Layer 가 가장 수정이 많음  
-> 가장 많이 바뀌는 쪽을 가장 위:주로 개발팀 (uxui)  
-> 레이어를 다 캐싱해 둠(서비스 외 필요 라이브러리는 다 두고 나옴)
- 내부에서는 레지스트리 잘 안 올림 내부에 따로 서버 지정 함  
  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3d018a90-96e9-4188-89ae-86faaabd5d50" />
  - 포트 퍼블리싱 (포트 매핑)
  - 
권장하지 않지만 네트워킹 구조는 이러함  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6b47e701-a423-49d9-8609-dc0a10a98f7e" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9fa72b4e-e7b6-4ee6-b325-85fed3f12760" />

S3는 아마존 데이터를 말 함 스토리지 구축에도 비용 많이 들어감

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/23aa87db-8ad2-49f8-a67a-cb469dab1c15" />

cpu와 메모리같은 I/O자원 중요

sudoers가 있음:설치한 사람-설치할 때 계정 아이디 물어볼 때 이미 등록되는 것(부하직원이랑 책임을 구분하고 싶을 때 sudoers에 넣어줌 etc/group에 있음 :내가 만든 마스터 아이디 딱 하나 뜸)<-> owner group guest 

나중에 앤서블로 복제해서 vm1.vm2,vm3 만드는 연습

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ed9f131b-d4ad-4a37-974f-d6378310f3e8" />
리소스 제안 하는 용도 많지 않음(쿠보네티스가 많이 하고 있음)
