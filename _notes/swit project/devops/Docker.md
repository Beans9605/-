```toc
```

[[Devops Study Index]]
데브옵스 목차 페이지

# Docker란 무엇인가
도커는 쿠버네티스와 관련된 여러 기능 제공
1. 컨테이너 이미지 포맷
2. 컨테이너 수명 주기 관리 컨터이너 런
3. 타임 라이브러리
4. 컨테이너 패키징 및 실행하는 command 도구
5. 컨테이너 관리용 API

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

## 컨테이너 런타임

	참고: Dockershim은 쿠버네티스 릴리즈 1.24 부터 쿠버네티스 프로젝트에서 제거됨.

파드가 노드에서 실행될 수 있도록 클러스터의 각 노드에 컨테이너 런타임을 설치해야함.
쿠버네티스 1.25에서는 컨테이너 런타임 인터페이스(CRI) 요구사항을 만족하는 런타임을 사용해야함.

### Docker Engine

- 참고: 아래의 지침은 도커 엔진과 쿠버네티스를 통합하는데 [cri-dockerd](https://github.com/Mirantis/cri-dockerd) 어댑터를 사용하고있다고 가정.

1. 각 노드에서, [도커 엔진 설치하기](https://docs.docker.com/engine/install/#server)에 따라 리눅스 배포판에 맞게 도커를 설치
2. [cri-dockerd](https://github.com/Mirantis/cri-dockerd) 소스 코드 저장소의 지침대로 cri-dockerd를 설치
cri-dockerd의 경우 CRI 소켓은 기본적으로 /run/cri-dockerd.sock 이다.

#### 샌드박스(pause) 이미지 덮어쓰기
cri-dockerd 어댑터는, 파드 인프라 컨테이너("pause image")를 위해 어떤 컨테이너 이미지를 사용할지 명시하는 커맨드라인 인자를 받음. 해단 커맨드라인 인자는 --pod-infra-container-image 임.
