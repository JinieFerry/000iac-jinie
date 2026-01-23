앤서블은 집이나 작은 회사에서 구현할 수 있는 것 -> 쿠보네티스는 자동화 되어있어서 개념을 알 수 없음     
다음주부터는 도커 들어갈 것   

구조화

한 페이지가 거의 같은 형식으로 예쁘게 나오는 게 지금이라면 이건 쓰든 안 쓰든 다 규격을 미리 정해두는 게 프레임 (옛날엔 자바스크립트로 스트레스)

자바 프레임은 무거워서 파이썬이 프레임을 만들었음- 나중에 경쟁이 될 것 같음 


로드밸런싱 작업1 !이름이 중!

# 하나 더 추가 될 것이고, hostvars가 빠져서 만들어줘야 하는 상태 (아래에서 hostvars사용함)
nginx-role-project/
├── inventory.yml                 # 전체 서버 그룹 및 IP 정의
├── deploy_site.yml               # 전체 시스템 구축용 메인 플레이북 #전에는 site하나가 전체 파일을 실행하게 했는데
├── handlers/                     # [통합] 전역 핸들러 보관함
│   └── main.yml                  # 모든 서버가 공유하는 통합 핸들러 (Listen 활용)
├── roles/                        # 역할별 로직 보관함
│   ├── webserver/
│   │   └── tasks/
│   │       └── main.yml          # 웹 서버 설치 및 index.html 생성
│   └── loadbalancer/
│       ├── tasks/
│       │   └── main.yml          # LB 설치 및 설정 배포 
│       └── templates/
│           └── lb_nginx.conf.j2  # LB 설정 템플릿



### 3. 주요 파일별 소스 코드
#### ① 통합 핸들러 파일 (handlers/main.yml)
어떤 서버(Web, LB)에서든 trigger_nginx_restart라는 이름으로 신호를 보내면, 각 서버 환경에 맞는 핸들러가 자동으로 동작합니다.

```YAML


---
# 전역 지능형 핸들러. 자동으로 그룹명과 서비스 명을 결합 restart 해줍니다.  : 좀 더 프로그래밍 감각
- name: "Universal Service Controller"
  ansible.builtin.service:
    name: "{{ (group_names | intersect(['web_servers', 'load_balancer'])) | length > 0 | ternary('nginx', 'mysql') }}" #intersect에 DB서버도 들어가고 늘어날 것
    state: "{{ 'reloaded' if 'web_servers' in group_names else 'restarted' }}"
  listen: "trigger_restart" #핸들러스에 맞춰서 tasks 웹서버 role,로드밸런서 role 수정함

# 또는 아래와 같이 작성하여 IaC 의 뜻대로 명시성을 확보할 수도 있습니다. : 읽기 쉽게
# 각 역할의 이름을 명확히 하되, listen 토픽을 통일하여 어디서든 호출 가능하게 함
- name: "Restart Web Services"
  ansible.builtin.service:
    name: nginx
    state: reloaded
  when: "'web_servers' in group_names" #지금 root방에 있어서 어디 볼 지를 모르니까 어딜 볼 지 알려주는 것 roles에서 웹서버 뜨면 작동해라
  listen: "trigger_restart"

- name: "Restart LB(loadbalancing) Services"
  ansible.builtin.service:
    name: nginx
    state: restarted
  when: "'load_balancer' in group_names"
  listen: "trigger_restart"

- name: "Restart DB Services"   # 향후 함께 구축하게될 DB서버
  ansible.builtin.service:
    name: mysql
    state: restarted
  when: "'db_servers' in group_names"
  listen: "trigger_restart"

```



#### ① 인벤토리 파일 (`inventory.yml`) : 웹서버 두 개를 할 꺼고 하나를 로드밸런서로 쓸 것

```yaml
all:
  children:
    load_balancer:
      hosts:
        lb_node:
          ansible_host: 192.168.10.100    # 로드밸런서 IP
    web_servers:
      hosts:
        web_01:
          ansible_host: 192.168.10.101    # 웹 서버 1 IP
        web_02:
          ansible_host: 192.168.10.102    # 웹 서버 2 IP

```

#### ② 메인 플레이북 (`deploy_site.yml`) 사이트를 시작하는게 대부분이라 사이트로 이름 지음 (웹서버에서는 인덱스)

```yaml
---
# 1. 웹 서버 역할 실행 인벤토리 불러주구
- name: 백엔드 웹 서버 계층 배포
  hosts: web_servers #webserver
  become: true
  roles:
    #위 트리구조 폴더명에 맞춰서 이 폴더명 수정함      
    - webserver                   

# 2. 로드 밸런서 역할 실행 롤스 불러오고
- name: 전면 로드 밸런서 계층 배포
  hosts: load_balancer # 인벤토리명이랑 일치해야 함
  become: true
  roles:
    #위 트리구조 폴더명에 맞춰서 이 폴더명 수정함
    - loadbalancer    # 디렉토리 명이랑 일치해야 함                    

# 전역 핸들러를 로드하여 모든 역할(Roles)에서 notify 신호를 수신함 핸들러는 이렇게 따로 불러야 함 (yml파일로 따로 묶어놔서)
  handlers:
    - import_tasks: handlers/main.yml #import는 아예 통째로 다 #include는 프로그램 중간 중간에 껴넣을 수 있는 것

```

#### ③ 웹 서버 역할 (`roles/webserver/tasks/main.yml`)

```yaml
---
- name: Nginx 웹 서버 패키지 설치
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true

- name: 서버별 개별 식별 페이지 생성
  ansible.builtin.copy:
    content: "<h1>Hello! This is {{ inventory_hostname }}</h1>" #호스트네임에 자기 번호 매겨서 아예 짓기 {{변수}}어디서 불렀지? 따로 지정해줘야 하는지 생각해보세요
    dest: /var/www/html/index.html
    mode: '0644'
  #위의(handlers/main.yml : listen: "trigger_restart")에 맞춰서 수정함 
  notify: "trigger_restart"  # 통합 핸들러 토픽 호출 #이름이 같아야 함 에러 날 것


```

#### ④ 로드 밸런서 역할 (`roles/loadbalancer/tasks/main.yml`)

```yaml
---
- name: Nginx 로드 밸런서 패키지 설치 #엔진엑스 서버가 로드발란서 역할을 할 것
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true

- name: 기본 설정 파일 제거
  ansible.builtin.file:
    path: /etc/nginx/sites-enabled/default  #sites-aviable 밑에 file로 link/
    state: absent #부재하게 해라 보통 도메인을 주는 자리에 nginx를 깔면 default가 나 옴 = 그 서비스 안 할거야 난 로드밸런싱 역할 할거야 그러니까 absent부재 해

- name: 로드 밸런싱 설정 템플릿 배포 #로드 발란싱은 설정파일(jinja2가 만들어 둠)이 필요함
  ansible.builtin.template:
    src: lb_nginx.conf.j2                # 같은 role 폴더 내 templates/에서 찾음
    dest: /etc/nginx/conf.d/load_balancer.conf
    mode: '0644'
  #(handlers/main.yml : listen: "trigger_restart")에 맞춰서 수정함 
  notify: "trigger_restart"  # 웹 서버와 동일한 토픽 이름 사용 #에러 날 것 뭐랑 일치 해야 하는 지 잘 보세요



```

#### ⑤ LB 설정 템플릿 (`roles/loadbalancer/templates/lb_nginx.conf.j2`) #모든 데몬 보면 컨프 파일이 있음

```nginx
# 로드발런싱 할 거니까 웹서버가 업스트림 #우린 두대만 정의했는데 개수 맞춰야 함 에러 날 것
# 자동화 처리
upstream my_backend_servers {
    {% for host in groups['web_servers'] %}    #{{}}:변수명, {% %}:제어문 for jinja2
    server {{ hostvars[host]['ansible_host'] }}:80 max_fails=3 fail_timeout=30s;
    {% endfor %}
}

# 자동화 처리 쓸 꺼면 이 아래처럼 작성은 주석처리 해야 함 둘 중 하나만 쓰는 것
# 자동화 하지 않고 명시적으로 남길 때는 아래처럼 작성합니다. 
upstream my_backend_servers {
    # 첫 번째 웹 서버: web-server-01 #위에서 두대 정의한 거랑 안 맞음 ip도 맞는지 확인해야함
    # max_fails와 fail_timeout을 명시하여 안정성 확보
    server 192.168.10.11:80 max_fails=3 fail_timeout=30s;  #max_fails=3:니가 서비스해~하고 데이터를 줬는데, 응답 줘~ 응답을 안주면 죽었는지 살았는지 3번만 체크하겠다 (응답 안 하면 다른 애한테 줄 거임) , 그리고 30초 안에 응답 안 하면 3번까지 안 물어 봄 #외국은 30초 주는데 한국은 더 짧게 조정함 (잠시 서비스 멈춰놓고 넘어가는지 확인가능)

    # 두 번째 웹 서버: web-server-02
    server 192.168.10.12:80 max_fails=3 fail_timeout=30s;

    # 세 번째 웹 서버: web-server-03
    server 192.168.10.13:80 max_fails=3 fail_timeout=30s;
}





# 아래 코드는 공통입니다.
# 로드밸런서 가상 서버 설정
server {
    listen 80; 
    server_name localhost; #nginx 돌릴 때 conf파일에서 썼던 내용들 

    location / {
        proxy_pass http://my_backend_servers; #중간 중개자를 proxy라고 부름 그래서 고객 클라이언트를 프록시로 알기 때문에
        proxy_set_header Host $host;  #사실 나 프록시 아니구 이사람이야~ 로그에 내 이름 말고 그 사람 이름 남겨야해~ 그래서 호스트 주소를 남기고
        proxy_set_header X-Real-IP $remote_addr; #실제 작업은 ip로 하기 때문에 실제 아이피를 남김 그래서 변수명 real ip라고 함
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; #그동안 거쳐온 애들한테 다 알려줘야 함 내 아이피 앞에 파이어월 아이피 붙고 마지막에 진짜 헤더를 붙여줘야 하는 것 - 네트웤 개념 그대로
        
        # 실제 처리된 백엔드 서버를 확인하기 위한 응답 헤더
        add_header X-Backend-Server $upstream_addr; # 되돌려 줄 때 응답할 때 이렇게 헤드 붙여서 나한테 줘 그럼 내가 그걸 가지고 보내줄게
    }
}

```


로드밸런싱 2,3에 실행 추가해뒀음

011.nginx 로드밸런싱2통합핸들러.md
https://github.com/putto4u/03.Ansible.IaC.InfraAuto/blob/main/0071.%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%AC%EC%8B%B1%20%EC%8B%A4%EC%8A%B5/011.nginx%20%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%8B%B12%ED%86%B5%ED%95%A9%ED%95%B8%EB%93%A4%EB%9F%AC.md

012.nginx 로드밸런싱3키관리추가.md
https://github.com/putto4u/03.Ansible.IaC.InfraAuto/blob/main/0071.%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%AC%EC%8B%B1%20%EC%8B%A4%EC%8A%B5/012.nginx%20%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%8B%B13%ED%82%A4%EA%B4%80%EB%A6%AC%EC%B6%94%EA%B0%80.md
