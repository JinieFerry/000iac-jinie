#### 04.PrivateCloudInfra/25. 실무구축Loose Coupling/
```
 1995  0210 putto
 ### 0025. fastAPI서버구축.md ###
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
 2007  kubectl apply -f fastapi-origin.yaml
 2008  kubectl apply -f fastapi-ori.yaml
 2009  k get pods -n wb-prod  -l app=fastapi -w
 2010  k apply -f fastaoi-ori.yaml 
 2011  k get pods -n wb-prod  -l app=fastAPI -W
 2012  k get pods -n wb-prod  -l app=fastAPI -w
 2013  kubectl get svc -n wb-prod fastapi-service
 2014  ls
```
```
 ### 0030. fastAPI용 pvc.md ###
 2015  vi fast-pvc.yaml
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
 2016  kubectl apply -f fast-pvc.yaml
```
브라우저 접속 확인: 192.168.11x.251:30001
<img width="910" height="1029" alt="image" src="https://github.com/user-attachments/assets/a09150de-d893-48bb-96ec-d6a2b0c972c7" />

```
 2017  vi fast-pvc.yaml 
 2018  kubectl get pvc back-pvc -n wb-prod
 2019  history
 2020  cd /opt
 2021  ls
 2022  cd local-path-provisioner/
 2023  ls
 2024  cd pvc-f5ab901c-cad1-4015-a340-f0d182d7cd3e_wb-prod_back-pvc/
 2025  ls
 2026  vi index.html 
 2027  cd k8s_lab
 2028  cd ..
```
 ### 0035. fast pvc 연결.md ###
 ```
 2029  cd
 2030  cd k8s_lab/
 2031  ls
 2032  cd 03_loosepj/
 2033  ls
 2034  vi fastapi-mount.yaml
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

```
```
