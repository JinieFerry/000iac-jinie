#### 04.PrivateCloudInfra/25. 실무구축Loose Coupling/
```
 1995  0210 putto
 ### 0025. fastAPI서버구축.md ###
 1996  k get nodes
 1997  k get pod
 1998  k get svc
 1999  k get deployments.apps 
 2000  k get ingress
 2001  k get namespaces 
 2002  cd k8s_lab/
 2003  ls
 2004  rm fast*
 2005  ls
 2006  vi fastaoi-ori.yaml
 2007  kubectl apply -f fastapi-origin.yaml
 2008  kubectl apply -f fastapi-ori.yaml
 2009  k get pods -n wb-prod  -l app=fastapi -w
 2010  k apply -f fastaoi-ori.yaml 
 2011  k get pods -n wb-prod  -l app=fastAPI -W
 2012  k get pods -n wb-prod  -l app=fastAPI -w
 2013  kubectl get svc -n wb-prod fastapi-service
 2014  ls
 ### 0030. fastAPI용 pvc.md ###
 2015  vi fast-pvc.yaml
 2016  kubectl apply -f fast-pvc.yaml
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
 ### 0035. fast pvc 연결.md ###
 2029  cd
 2030  cd k8s_lab/
 2031  ls
 2032  cd 03_loosepj/
 2033  ls
 2034  vi fastapi-mount.yaml
 2035  history ~11:20~

```
