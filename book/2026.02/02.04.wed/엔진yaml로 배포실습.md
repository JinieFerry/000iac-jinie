원본참고
>04.PrivateCloudInfra/21.exec/010. 엔진x 야믈로 배포1.md
>https://github.com/putto4u/04.PrivateCloudInfra/blob/main/21.exec/010.%20%EC%97%94%EC%A7%84x%20%EC%95%BC%EB%AF%88%EB%A1%9C%20%EB%B0%B0%ED%8F%AC1.md
>04.PrivateCloudInfra/21.exec/011.NFS iac로 구축하기.md
>https://github.com/putto4u/04.PrivateCloudInfra/blob/main/21.exec/011.NFS%20iac%EB%A1%9C%20%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0.md

삭제할 때는 하나 위의 단계를 삭제하면 완전하게 삭제 됨 (예시: pod를 삭제해서 삭제되지않으면 deployment를 삭제)   
실무에서는 구축배포와 블루/그린배포를 따로 함

#### 작업 지시서
우리 실습은 항상 최종은 프론트 웹서버가 있고, 뒤에 백엔드 웹서버가 있음
+ 백엔드는 db서버가 있고, 연동이 되어야 함:데이터가 있어야 해서 sql 주입, 스토리지 서버가 있어야함
+ db는 db서버 접속, 파일은 스토리지 서버 접속이 가능해야 함
+ 브라우저로 현상황을 보고

010. 엔진x 야믈로 배포1.md: python 3.9-slim (lts:latest 안정된 최신버전 파이썬웹으로 배포) - 스토리지 만들기
011.NFS iac로 구축하기.md: nginx,jango:
020.PV와 PVC 구현.md: - k8s랑 pv로 연결, pvc로 요청해서 생성하는 것 확인

