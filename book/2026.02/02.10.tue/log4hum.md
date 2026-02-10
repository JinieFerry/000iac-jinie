https://github.com/putto4u/04.PrivateCloudInfra/tree/main/25.%20%EC%8B%A4%EB%AC%B4%EA%B5%AC%EC%B6%95Loose%20Coupling    
본 실습에서는 FastAPI 애플리케이션을 Kubernetes Deployment로 구성하고, Service(NodePort)를 통해 Pod와의 결합도를 낮췄다.
또한, PersistentVolumeClaim을 활용하여 애플리케이션 컨테이너와 데이터 저장소를 분리함으로써 Loose Coupling 구조를 실무 환경에서 검증하였다.    
# 04.PrivateCloudInfra/25. 실무구축Loose Coupling/
 ### 0025. fastAPI서버구축.md ###
```
# vmmaster1 - a
   k get nodes
   k get pod
   k get svc
   k get deployments.apps 
   k get ingress
   k get namespaces 
   cd k8s_lab/
   ls
   rm fast*
   ls
   vi fastaoi-ori.yaml
```
+ fastaoi-ori.yaml
```

---
# [2] Deployment: 오리진 애플리케이션 (3 Replicas)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi            # [Simple] 이름을 단순화
  namespace: wb-prod       # [Prod] 운영 네임스페이스 적용
  labels:
    app: fastapi
spec:
  replicas: 3              # [Availability] 3개의 복제본 유지
  selector:
    matchLabels:
      app: fastapi
  template:
    metadata:
      labels:
        app: fastapi
    spec:
      containers:
      - name: fastapi-container
        image: tiangolo/uvicorn-gunicorn-fastapi:python3.9
        ports:
        - containerPort: 80

---
# [3] Service: 약한 결합의 접속점
apiVersion: v1
kind: Service
metadata:
  name: fastapi-svc
  namespace: wb-prod
spec:
  selector:
    app: fastapi           # [Loose Coupling] 라벨을 통해 파드들을 자동 연결
  ports:
    - protocol: TCP
      port: 80             # 내부 포트
      targetPort: 80       # 컨테이너 포트
      nodePort: 30001      # 외부 접속 포트
  type: NodePort
```
```
# vmmaster1 - a
   kubectl apply -f fastapi-origin.yaml
   kubectl apply -f fastapi-ori.yaml
   k get pods -n wb-prod  -l app=fastapi -w
   k apply -f fastaoi-ori.yaml 
   k get pods -n wb-prod  -l app=fastAPI -W
   k get pods -n wb-prod  -l app=fastAPI -w
   kubectl get svc -n wb-prod fastapi-service
   ls
```
```
 ### 0030. fastAPI용 pvc.md ###
# vmmaster1 - a
   vi fast-pvc.yaml
```
+ fast-pvc.yaml
```
# fast-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: back-pvc
  namespace: wb_prod
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path  # 설치된 Local Path Provisioner 활용
  resources:
    requests:
      storage: 1Gi              # 백엔드 데이터 저장용 2GB 요청

```
```
# vmmaster1 - a
   kubectl apply -f fast-pvc.yaml
```
브라우저 접속 확인: 192.168.11x.251:30001
<img width="910" height="1029" alt="image" src="https://github.com/user-attachments/assets/a09150de-d893-48bb-96ec-d6a2b0c972c7" />

```
# vmmaster1 - a
   vi fast-pvc.yaml 
   kubectl get pvc back-pvc -n wb-prod
   history
   cd /opt
   ls
   cd local-path-provisioner/
   ls
   cd pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc/
   ls
   vi index.html 
   cd k8s_lab
   cd ..
```
 ### 0035. fast pvc 연결.md ###
 ```
# vmmaster1 - a
   cd
   cd k8s_lab/
   ls
   cd 03_loosepj/
   ls
   vi fastapi-mount.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi           # 기존 Deployment와 동일한 이름 (수정 모드)
  namespace: wb-prod      # [필수] 동일한 네임스페이스
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fastapi
  template:
    metadata:
      labels:
        app: fastapi
    spec:
      containers:
      - name: fastapi-container
        image: tiangolo/uvicorn-gunicorn-fastapi:python3.9
        ports:
        - containerPort: 80
        
        # [핵심 1] 컨테이너 내부 경로 설정
        volumeMounts:
        - name: data-storage       # 아래 volumes에서 정의한 이름과 일치해야 함
          mountPath: /app/data     # 컨테이너 내부에서 데이터가 저장될 경로
          
      # [핵심 2] 사용할 스토리지(PVC) 지정
      volumes:
      - name: data-storage         # 볼륨의 별칭 (자유롭게 지정)
        persistentVolumeClaim:
          claimName: back-pvc      # 앞서 생성한 PVC 이름 (정확히 일치해야 함)
```
 2035  history ~11:20~

+ after k get svc check each 30001/31080/30080
<img width="910" height="1021" alt="image" src="https://github.com/user-attachments/assets/18cf5e3b-449a-4e64-9a76-d208e1e2b169" />
<img width="915" height="1030" alt="image" src="https://github.com/user-attachments/assets/85cf6acd-b883-4fe2-9a27-9e5f002d5e55" />
<img width="916" height="1026" alt="image" src="https://github.com/user-attachments/assets/8b557a45-65df-4e5e-96b4-fc276dee2534" />

### 0035. fast pvc 연결.md ###
```
# vmmaster1 - a
   mv fastapi-mount.yaml fastapi-mount.yml
   k apply -f fastapi-mount.yml 
   k get pods -n wb-prod 
   kubectl exec -it fastapi-6648b9b789-j27fj -n wb-prod -- /bin/sh
   ls
   cd
   cd /opt
   ls
   cd local-path-provisioner/
   ls
   cd
   ls
   cd k8s_lab/
   cd 03_loosepj/
   k get pod -o wide
   k get deployments.apps 
   k get deployments.apps -o wide
   k get svc
   k exec -it fastapi-6648b9b789-j27fj 
   ### open another vmmaster1 tab : vmmaster1 - b ###
```
   ### that another vmmaster1 : vmmaster1 - b ###
   ```
# vmmaster1 - b
   cd /opt
   ls
   cd local-path-provisioner/
   ls
   cd pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc/
   ls
   touch imsi
   ls
   ```

 2058  history ~11:56~ 
   ### go to opend another vmmaster1 tap: vmmaster1 - b ###
   ### came here:opend another vmmaster1 tap: vmmaster1 - b  ###
```
# vmmaster1 - b
master@vmmaster1:/opt/local-path-provisioner/pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc$ vi main.py
````
 ### create your own index.html code with ai and test it ###
 ```
# vmmaster1 - b
master@vmmaster1:/opt/local-path-provisioner/pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc$ vi main.py
master@vmmaster1:/opt/local-path-provisioner/pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc$ ls
imsi  index.html  main.py
master@vmmaster1:/opt/local-path-provisioner/pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc$
```
### go back to first vmmaster1 tap:  vmmaster1 - a ###
### came back here: first vmmaster1 tap:  vmmaster1 - a ###
   ```
# vmmaster1 - a
   cd
   cd /opt
   ls
   cd local-path-provisioner/
   ls
   cp main.py ./pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc/
   ls
   cd pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc/
   ls
   nano main.py
   history
```
 2119  ~~~ 12:26 ~~~
 2120  history: index.html을 #/app/data로 이동시키기 위해서 (ai로 만든 근사한 사이트로 브라우저 접속 시 보이도록 적용시키려고 index.html을 이동)
fastapi는 main.py를 가장 먼저 실행시킴 index.html이 아니라 그래서 다시 ai로 근사한 html을 만들어서 main.py 파일을 작성하고 브라우저에 뜨는 디렉토리로 카피함.      

```
# vmamster1 - a
# try: change the mountpoint putto alone
```
##### delete cache and try 192.168.11x.251:30001 and check but not updated : failed ######

==========    
LUNCH TIME   
==========

#### ~~~ vmmaster1 - a : 14:30 ~~~ ####
```
   cd 
   cd k8s_lab/
   ls
   mv fastaoi-ori.yaml fastapi-ori.yaml
   k delete deployments.apps fastapi
   k get pod
   cd /opt
   cd local-path-provisioner/
   ls
   cd pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc/
   ### go to 1b ###
```
####  ~~~ vmmaster1 - b : 14:30 ~~~ ####
``` 
   ### i am 1b ###
   vi fastapi-mount.yml 
   k get deployments.apps 
   k get pod
   ls
   k apply -f fastapi-mount.yml 
   k get pods
   vi fastapi-mount.yml 
   ## (ch) mountPath: /app/data -> /app ##
```

  # jinie try ㅣ sscc ! #
  ```
   vi fastapi-mount.yml 
   k apply -f fastapi-mount.yml
   k rollout restart deploy/fastapi -n wb-prod
   k get pods -n wb-prod -l app=fastapi -w
   vi fast-pvc.yaml 
   k exec  -it fastapi-b4d8ff556-n6xcv -n wb-prod -- /bin/sh
   k get pods -n wb-prod
   k exec -ir fastapi-56868c5b94-wmdlc -n wb-prod -- /bin/sh
   kubectl get pod fastapi-56868c5b94-wmdlc -n wb-prod -o jsonpath='{.spec.containers[*].name}{"\n"}'
   kubectl exec -it fastapi-56868c5b94-wmdlc -n wb-prod -c fastapi-container -- /bin/sh
   kubectl get pod fastapi-56868c5b94-wmdlc -n wb-prod -o wide
   kubectl get pod fastapi-56868c5b94-wmdlc -n wb-prod -o jsonpath='{.status.phase}{"\n"}{.status.containerStatuses[*].state}{"\n"}'
   kubectl describe pod fastapi-56868c5b94-wmdlc -n wb-prod | tail -n 60
   kubectl logs fastapi-56868c5b94-wmdlc -n wb-prod -c fastapi-container --previous --tail=200
   cd /opt/local-path-provisioner
   ls
   cd pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc/
   ls -al
   cat > /opt/local-path-provisioner/pvc-*_wb-prod_back-pvc/main.py <<'PY'
from fastapi import FastAPI
from fastapi.responses import HTMLResponse
from pathlib import Path

app = FastAPI()

@app.get("/", response_class=HTMLResponse)
def root():
    p = Path("/app/index.html")
    if p.exists():
        return p.read_text(encoding="utf-8")
    return "<h1>index.html not found in /app</h1>"
PY
```

```
   ls -al
   kubectl rollout restart deploy/fastapi -n wb-prod
   kubectl get pods -n wb-prod -l app=fastapi -w
```
+ index.html을 main.py이 불러서 실행하므로 index.html을 수정해서 같은 주소로 브라우저에서 접속시 (새로고침) 수정이 적용되는지 확인하기
+ 192.168.11x.251:30001

```
# caty으로 index.html 수정함
   cat > index.html <<'HTML'
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>TRASH TO FRESH — PINK BUILD</title>
  <style>
    :root{
      --bg1:#ff4fd8;
      --bg2:#ff86ea;
      --bg3:#ffd1f6;
      --ink:#1a0b1a;
      --card:rgba(255,255,255,.14);
      --stroke:rgba(255,255,255,.28);
      --shadow: 0 20px 60px rgba(20,0,20,.25);
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0;
      font-family: ui-sans-serif, system-ui, -apple-system, "Apple SD Gothic Neo","Noto Sans KR", Segoe UI, Roboto, Arial, sans-serif;
      color:rgba(255,255,255,.92);
      background:
        radial-gradient(1200px 800px at 20% 10%, rgba(255,255,255,.25), transparent 60%),
        radial-gradient(900px 700px at 80% 20%, rgba(255,255,255,.16), transparent 55%),
        radial-gradient(900px 900px at 40% 90%, rgba(0,0,0,.18), transparent 60%),
        linear-gradient(135deg, var(--bg1), var(--bg2) 55%, var(--bg3));
      overflow:hidden;
    }
    .noise{
      position:fixed; inset:0;
      background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='180' height='180'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.9' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='180' height='180' filter='url(%23n)' opacity='.22'/%3E%3C/svg%3E");
      mix-blend-mode:soft-light;
      pointer-events:none;
    }
    .wrap{
      position:relative;
      height:100%;
      display:grid;
      grid-template-rows:auto 1fr auto;
      gap:20px;
      padding:28px;
      max-width:1100px;
      margin:0 auto;
    }
    header{
      display:flex;
      justify-content:space-between;
      align-items:center;
      letter-spacing:.28em;
      text-transform:uppercase;
      font-size:12px;
      opacity:.95;
    }
    .status{
      display:flex; align-items:center; gap:10px;
      font-size:12px; letter-spacing:.08em; text-transform:none;
      background:rgba(0,0,0,.18);
      border:1px solid rgba(255,255,255,.18);
      padding:10px 12px;
      border-radius:999px;
      box-shadow: 0 8px 22px rgba(0,0,0,.15);
    }
    .dot{
      width:9px; height:9px; border-radius:999px;
      background:#00ff9a;
      box-shadow:0 0 0 6px rgba(0,255,154,.18);
      animation:pulse 1.6s ease-in-out infinite;
    }
    @keyframes pulse{
      0%,100%{transform:scale(1); opacity:1}
      50%{transform:scale(1.15); opacity:.75}
    }
    .grid{
      display:grid;
      grid-template-columns: 1.1fr .9fr;
      gap:18px;
      align-items:stretch;
      min-height:0;
    }
    .card{
      background:var(--card);
      border:1px solid var(--stroke);
      border-radius:22px;
      box-shadow: var(--shadow);
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      overflow:hidden;
      position:relative;
    }
    .card .inner{ padding:22px; }
    .title{
      font-size:40px;
      line-height:1.05;
      margin:0 0 12px 0;
      letter-spacing:-.02em;
      color:rgba(255,255,255,.96);
      text-shadow:0 10px 35px rgba(0,0,0,.22);
    }
    .subtitle{
      margin:0 0 18px 0;
      opacity:.9;
      font-size:14px;
      line-height:1.7;
      max-width:58ch;
    }
    .pillrow{display:flex; flex-wrap:wrap; gap:10px; margin-top:12px;}
    .pill{
      padding:8px 12px;
      border-radius:999px;
      background:rgba(0,0,0,.16);
      border:1px solid rgba(255,255,255,.18);
      font-size:12px;
      letter-spacing:.03em;
      user-select:none;
    }
    .chat{
      display:flex;
      flex-direction:column;
      gap:12px;
      min-height:0;
    }
    .bubble{
      align-self:flex-start;
      max-width:86%;
      padding:14px 14px;
      border-radius:16px;
      background:rgba(0,0,0,.16);
      border:1px solid rgba(255,255,255,.16);
      box-shadow: 0 10px 30px rgba(0,0,0,.14);
      line-height:1.55;
      font-size:13px;
      white-space:pre-line;
    }
    .bubble.me{ align-self:flex-end; background:rgba(255,255,255,.14); }
    .meta{
      font-size:12px;
      opacity:.85;
      display:flex;
      justify-content:space-between;
      margin-bottom:10px;
    }
    .input{
      display:flex;
      gap:10px;
      padding:14px;
      border-top:1px solid rgba(255,255,255,.14);
      background:rgba(0,0,0,.10);
    }
    input{
      width:100%;
      padding:12px 14px;
      border-radius:14px;
      border:1px solid rgba(255,255,255,.20);
      background:rgba(0,0,0,.18);
      color:white;
      outline:none;
    }
    button{
      padding:12px 14px;
      border-radius:14px;
      border:1px solid rgba(255,255,255,.22);
      background:rgba(255,255,255,.18);
      color:white;
      cursor:pointer;
    }
    button:hover{ background:rgba(255,255,255,.24); }
    footer{
      opacity:.85;
      font-size:12px;
      display:flex;
      justify-content:space-between;
      gap:12px;
    }
    .build{
      font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
      opacity:.9;
    }
    @media (max-width:900px){
      .grid{grid-template-columns:1fr; }
      .title{font-size:34px;}
    }
  </style>
</head>
<body>
  <div class="noise"></div>
  <div class="wrap">
    <header>
      <div>TRASH · TO · FRESH</div>
      <div class="status"><span class="dot"></span>connected · pink build</div>
    </header>

    <div class="grid">
      <section class="card">
        <div class="inner">
          <h1 class="title">PINK UPGRADE</h1>
          <p class="subtitle">
            이 페이지는 “배포된 파일”이 아니라 <b>PVC에 마운트된 파일</b>이다.<br/>
            그래서 <b>index.html</b>을 바꾸면, <b>kubectl apply 없이</b>도 화면이 즉시 바뀐다.
          </p>
          <div class="pillrow">
            <div class="pill">NodePort :30001</div>
            <div class="pill">PVC back-pvc</div>
            <div class="pill">mountPath /app</div>
            <div class="pill">APP_MODULE main:app</div>
          </div>
        </div>
      </section>

      <section class="card chat">
        <div class="inner" style="padding-bottom:10px;">
          <div class="meta">
            <div>chatroom</div>
            <div class="build">BUILD · PINK · 2026-02-10</div>
          </div>

          <div class="bubble">여긴 핑크 테스트방이야.  
한눈에 “업데이트 됐다”는 걸 보여줄 거야.</div>

          <div class="bubble">바로 지금, 너는 <b>Loose Coupling</b>을 증명하고 있어.  
Pod는 바뀌어도, 화면은 PVC를 본다.</div>

          <div class="bubble me">내가 index.html만 바꿨는데도  
브라우저가 바뀌는 게 진짜네.</div>
        </div>

        <div class="input">
          <input value="이건 전송되지 않아. 그냥 흔적만 남아." readonly />
          <button type="button" onclick="blink()">send</button>
        </div>
      </section>
    </div>

    <footer>
      <div>TIP: 새로고침(F5). 캐시가 의심되면 Ctrl+F5.</div>
      <div class="build" id="stamp"></div>
    </footer>
  </div>

  <script>
    const stamp = document.getElementById('stamp');
    stamp.textContent = "loaded · " + new Date().toLocaleString();
    function blink(){
      document.body.style.filter = "saturate(1.3) contrast(1.05)";
      setTimeout(()=>document.body.style.filter="", 180);
    }
  </script>
</body>
</html>
HTML
```
```
# 롤아웃하고 적용
k rollout restart deploy/fastapi  -n wb-prod

```
+ index.html 수정 전
<img width="857" height="904" alt="image" src="https://github.com/user-attachments/assets/4ce28dfe-f5f0-446c-be67-470d4fde9d1e" />


+ index.html 수정 후 (블랙->핑크) , main.py으로 fastapi 돌리므로 fastapi-mount.yaml or .yml에서 env 선언 (main,app)
<img width="917" height="1030" alt="image" src="https://github.com/user-attachments/assets/48b36625-587e-48b1-a2f6-68a65d97bf88" />

### 0036. 세 웹엔진 pvc 연결경로.md ###



