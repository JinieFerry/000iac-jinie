# 실습 교재: 04.PrivateCloudInfra/325. 실무구축 Loosecoupling
2026.02.11 – Helm / Prometheus / Ingress 정리 로그       

```
   ### 04.25.40 ingress ###
   sed -n '1,120p' wb-ingress.yml
   vi wb-ingress.yml # host:지우고 -http:로 시작하게 하기
   k apply -f wb-ingress.yml # 설정 적용
   k get ingress -n wb-prod  # 인그레스 상태 확인

```
+ wb-ingress.yml
```
  1 # wb-ingress.yml                                                                  
  2 apiVersion: networking.k8s.io/v1
  3 kind: Ingress
  4 metadata:
  5   name: wb-ingress
  6   namespace: wb-prod
  7   annotations:
  8     kubernetes.io/ingress.class: "nginx"
  9     # 경로 끝의 '/' 처리를 위한 설정 (Rewrite)
 10     nginx.ingress.kubernetes.io/rewrite-target: /
 11 spec:
 12   # 로드밸런서가 없는 환경에서 고정 IP를 명시적으로 사용하는 규칙
 13   rules:
 14   - http:
 15       paths:
 16       - path: /
 17         pathType: Prefix
 18         backend:
 19           service:
 20             name: fr-end
 21             port:
 22               number: 80
 23       - path: /vew
 24         pathType: Prefix
 25         backend:
 26           service:
 27             name: py-service
 28             port:
 29               number: 80
 30       - path: /bucket
 31         pathType: Prefix
 32         backend:
 33           service:
 34             name: fastapi-svc
 35             port:
 36               number: 80

```

+ externalIPs 직접 먹여서 해결 : 192.168.251/까지만 쳐도 접속 가능 + /view , /bucket
```
 2216  kubectl get svc -n ingress-nginx ingress-nginx-controller -o yaml | egrep -n "type:|externalIPs|loadBalancerIP|ports:|nodePort" -n
 2217  kubectl -n ingress-nginx patch svc ingress-nginx-controller   -p '{"spec":{"externalIPs":["192.168.115.251"]}}'
 2218  kubectl get svc -n ingress-nginx ingress-nginx-controller -o wide

```

<img width="917" height="1030" alt="image" src="https://github.com/user-attachments/assets/7a6a37aa-4b31-4e4c-850b-d0ffbc489021" />
<img width="917" height="1030" alt="image" src="https://github.com/user-attachments/assets/995d7d09-7afd-4606-b34d-3cc52d09fad9" />
<img width="917" height="1030" alt="image" src="https://github.com/user-attachments/assets/540e2fe8-77f3-4d8a-a6ef-aa3b1c987086" />
