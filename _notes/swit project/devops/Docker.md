# Docker란 무엇인가
도커는 쿠버네티스와 관련된 여러 기능 제공
1. 컨테이너 이미지 포맷
2. 컨테이너 수명 주기 관리 컨터이너 런타임 라이브러리
3. 컨테이너 패키징 및 실행하는 command 도구
4. 컨테이너 관리용 API

# Dockerfile
``` Dockerfile
FROM golang:1.11-alpine AS build
WORKDIR /src/
COPY main.go go.* /src/
RUN CGO_ENABLED=0 go build -o /bin/demo

FROM scratch
COPY --from=build /bin/demo /bin/demo
ENTRYPOINT ["/bin/demo"]
```

**멀티 스테이지 빌드** 가 적용된 Dockerfile
