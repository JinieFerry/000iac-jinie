
인증서 삭제하고 서버로 다시
```
# CN을 server.putto.ai.kr로 지정하고 SAN에 도메인 추가
./easyrsa --subject-alt-name="DNS:putto.ai.kr" build-server-full server.putto.ai.kr nopass
```

