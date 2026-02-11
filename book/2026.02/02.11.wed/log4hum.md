# 실습 교재: 04.PrivateCloudInfra/325. 실무구축 Loosecoupling

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
