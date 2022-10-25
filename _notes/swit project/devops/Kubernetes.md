```toc
```

[[Devops Study Index]]
Devops 목차 페이지

#kubernetes #service #deployment #serviceaccount #clusterRoleBinding

출처 - 쿠버네티스를 활용한 클라우드 네이티브 데브옵스

# Kubernetes
상용 워크로드를 위한 대규모 컨테이너 오케스트레이터
서버 업그레이드, 보안 패치 설치, 네트워크 구성, 백업과 같은 전통적인 시스템 관리 작업 대부분은 클라우드 네이티브에서는 그다지 중요하지 않음, 쿠버네티스는 이러한 작업을 자동화.

## 장점
1. 쿠버네티스는 다중화와 절체 기능이 내장되어 애플리케이션의 안정성과 탄력성을 제공
2. 일부 관리형 서비스는 수요에 맞게 쿠버네티스 클러스터 자체를 확장 및 축소할 수 있어서 특정 시점에 필요한 것보다 더 큰 클러스터 비용을 지불하지 않아도 됨. (오토 스케일링)
3. 스케일링, 로드 밸런싱, 절체 기능 제공

## 단점
쿠버네티스가 만능은 아님, 레플리카를 만들어서 사용하는 방식으로는 db를 호환하지 않으니, 데이터베이스는 고유한 상태를 갖고 있으며 데이터베이스 레플리카를 배포하려면 다른 노드와의 조정이 필요하고 동시에 스키마 변경과 같은 작업이 모든 곳에서 수행돼야하는 단점 존재.
- db는 container로 관리할 수 없음

## 클라우드 컴퓨팅 마무리
클라우드 컴퓨팅은 하드웨어 관리에 따른 비용과 오버헤드를 없애고, 탄력적이고 유연하며 확장 가능한 분산 시스템을 구축할 수 있게 도와준다.  

- **데브옵스**는 현대의 소프트웨어 개발이 단순히 완성한 코드를 전달하는 데서 멈추지 않는다는 인식에서 출발한다. 데브옵스는 코드를 작성하는 사람과 코드를 사용하는 사람 사이에서 지속적으로 피드백을 주고받는 것이다. 
- **데브옵스**는 또한 인프라 및 운영 세계에서 코드 중심 접근 방식과 우수한 소프트웨어 엔지니어링 방법을 제공한다. 
- **컨테이너**는 작고 표준화된 독립적인 단위로 소프트웨어를 배포하고 실행할 수 있다. 따라서 컨테이너화된 마이크로서비스를 서로 연결하면 크고 다양한 분산 시스템을 쉽고 저렴하게 구축할 수 있다. 
- **오케스트레이터**는 컨테이너 배포, 스케줄링, 스케일링, 네트워킹을 담당하며, 우수한 시스템 관리자가 할 수 있는 모든 작업을 자동화와 프로그래밍화가 가능한 방식으로 처리 한다. 
- 쿠버네티스는 사실상 표준 컨테이너 오케스트레이터로 언제든 상용 환경에서 바로 사용할 준비가 되어 있다. 
- **클라우드 네이티브**는 클라우드 기반의 컨테이너화된 분산 시스템이다. 클라우드 네이티브는 자동화된 코드형 인프라로 동적으로 관리되는 마이크로서비스로 구성된다. 
- 운영과 인프라 기술은 클라우드 네이티브 혁명에 의해 쓸모없어지기는커녕 그 어느 때보다 **더 중요해질 것이다.** 
- **중앙 팀**은 다른 모든 팀에서 데브옵스를 가능하게 하는 플랫폼과 도구를 구축하고 관리해야 한다.

# Kubernetes 실습

## 컨테이너 이미지 실행

### 컨테이너 이미지란?
- 압축 파일로 비유할 수 있음
- 바이너리 파일이며 고유 ID를 가짐
- 컨테이너 실행에 필요한 모든 것을 담고 있음.

### 이미지 실행
컨테이너를 도커로 직접 실행하던지 쿠버네티스 클러스터에서 실행하던지 컨테이너 이미지에 ID나 URL을 지정해야함.

``` bash
$ docker container run -p 9999:8888 --name hello cloudnatived/demo:hello
```

위 코드를 실행하기 위해 데모 애플리케이션을 다운로드, 시퀀스는 아래에 참조

```Shell
$ git version
```

```Shell
$ git clone https://github.com/cloudnativedevops/demo.git
Cloning into demo...
...
```

## 컨테이너 빌드하기

컨테이너 이미지는 컨테이너 실행에 필요한 모든 것이 담긴 단일 파일
- 컨테이너 이미지 빌드는 docker image build 명령어를 사용
- 이 명령어는 Dockerfile 이란 텍스트 파일을 입력으로 받음
- 기존 이미지를 기반으로 새로운 이미지를 빌드 가능

```shell
$ docker image build -t myhello .
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM golang:1.11-alpine AS build
...
Successfully built eeb7d1c2e2b7
Successfully tagged myhello:latest
```

[[Docker#Dockerfile]]


# 쿠버네티스 구축하기
쿠버네티스는 클라우드 네이티브 세계의 운영 체제로 컨테이너화된 워크로드를 안정적이고 확장 가능하게 실행 할 수 있는 플랫폼 제공

## 쿠버네티스 운영법
1. 클라우드 인스턴스나 베어 메탈 서버에 직접 쿠버네티스 구축하고 운영
2. 관리형 쿠버네티스 서비스 사용 (AWS, naverCloud 이하 NCP)
3. 워크플로 도구, 대시보드, 웹 인터페이스를 활용하여 확장 가능한 관리형 플랫폼 사용
기타 등등의 방법이 있음

- 초보적인 관점에서 클러스터 구축, 튜닝, 트러블 슈팅과 같은 쿠버네티스의 기술적인 세부 사항에 대해서 다루지 않기로 한다. (고급편 쿠버네티스 바이블을 참조하여 작성 예정)

## 클러스터 아키텍처
쿠버네티스는 여러 대의 서버가 하나의 클러스터로 연결되어 있음.

### 컨트롤 플레인
컨트롤 플레인은 클러스터의 두뇌 역할을 하며 컨테이너 스케줄링, 서비스 관리, API 요청 처리 등의 작업을 수행
<img src="/assets/Pasted%20image%2020221017191828.png" />
![](Pasted%20image%2020221017191828.png)
컨트롤 플레인은 다음과 같은 컴포넌트로 구성됨

#### kube-apiserver
컨트롤 플레인의 프론트엔드 서버로 API 요청을 처리

#### etcd
어떤 노드가 존재하고 클러스터에 어떤 리소스가 존재하는지와 같은 쿠버네티스와 관련된 모든 정보를 저장하는 데이터베이스

#### kube-scheduler
새로 생성된 pod를 실행할 노드를 결정

#### kube-controller-manager
디플로이먼트와 같은 리소스 컨트롤러를 관리

#### cloud-controller-manager
클라우드 기반 클러스터는 클라우드 업체와 연동하여 로드 밸런서나 디스크 볼륨과 같은 자원을 관리

컨트롤 플레인 컨포넌트는 클러스터 내 **마스터 노드**에서 실행

### 노드 컴포넌트
워커 노드는 클러스터 내에서 사용자의 **워크로드**를 실행, 쿠버네티스 클러스터의 각 워커 노드는 다음과 같은 컴포넌트 실행

#### kubelet
노드에 예약된 워크로드를 실행하기 위해 컨테이너 런터임을 관리하고 상태를 모니터링

#### kube-proxy
서로 다른 노드에 있는 파드 간 통신이나 파드와 인터넷 사이의 네티워크 트래픽을 라우팅

#### container-runtime
컨테이너를 시작하고 중지하며 컨테이너 간 통신을 처리, 일반적으로 도커가 사용됨 (220501 이후로 도커의 공식 지원이 중단된 상태)
rkt나 cri-o와 같은 다른 컨타임 런타임을 공식지원

서로 다른 소프트웨어 컴포넌트를 실행하는 것 외에는 마스터 노드와 워커 노드 사이에 본질적인 차이는 없음
마스터 노드는 일반적으로 사용자 워크로드를 실행하지 않음.
<img src="/assets/Pasted%20image%2020221017192549.png" />
![](Pasted%20image%2020221017192549.png)

### 고가용성
올바르게 구성된 쿠버네티스 컨트롤 플레인은 마스터 노드로 여러 개로 구성되어 **고가용성**을 보장.
개별 마스터 노드에 장애가 발생하거나 컨트롤 플레인 컴포넌트가 중지되어도 클러스터는 정상적으로 작동.
또한 마스터 노드는 정상 작동하지만 네트워크 장애 때문에 일부 컴포넌트가 서로 통신하지 못하는 **네트워크 파티션** 상황에서도 컨트롤 플레인의 고가용성으로 이를 처리 가능

- etcd 데이터베이스는 여러 노드에 걸쳐 복제
	- 복제본의 절반 이상이 사용 가능한 상태라면 개별 etcd 노드 장애에도 고가용성 보장

#### 컨트롤 플레인 장애
사용자의 애플리케이션을 다운 시키지는 않지만 이상하고 잘못된 작동을 유발

**예시**
- 클러스터의 모든 마스터 노드를 중지할 경우 워커 노드의 파드는 한동안 계속 실행
- 새로운 컨테이너를 배포하거나 쿠버네티스 리소스를 변경 불가, 디플로이먼트와 같은 컨트롤러의 중단

그러므로 컨트롤 플레인의 고가용성은 클러스터가 올바르게 작동하기 위해 중요함.
하나의 노드에서 장애가 발생하더라도 클러스터가 쿼럼을 유지할 수 있도록 충분한 수의 마스터 노드가 필요
상용 클러스터의 경우 마스터 노드가 최소 3개 필요 ( 상용 구조 참조 )

#### 워커 노드 장애
워커 노드 장애는 별로 중요한 문제는 아님.

- 컨트롤 플레인이 작동하고 있다면 쿠버네티스가 워커 노드의 장애를 감지하고 해당 노드의 파드를 다른 노드로 재조정할 것
- 대규모 노드 장애가 한 번에 발생하면 클러스터에는 워크로드를 실행할 수 있는 충분한 자원이 없어짐 -> 자주 일어나는 일은 아님 -> 일어나더라도 노드를 복구하는 동안 쿠버네티스가 가능한 많은 파드를 실행
	- 자원 부족에 따른 Notification Service를 구축해두면 장애 컨트롤이 쉬워짐

#### 검증은 필수
고가용성은 마스터 노드 하나 혹은 워커 노드 몇 개에 장애가 발생하더라도 클러스터에는 문제가 없도록 보장,

**테스트 조건**
- 실제로 그러한가 테스트
- 시스템 점검 작업이나 트래픽이 적은 시간대에 워커 노드를 재부팅 후 상태 점검
- **더 까다로운 테스트** 
	- 마스터 노드 재부팅 (클라우드 기반에선 불가능 (마스터 노드를 클라우드에서 구성))

상용 등급이라면 위 전제 조건에서 이상이 없어야함

#### 자체 호스팅 쿠버네티스 비용
쿠버네티스에서 상용 워크로드 실행을 고려하는 사람들이 직면하는 가장 어려운 결정은 **'구축'** 할 것인가 **'구입'** 할 것인가

1. 자체 호스팅 쿠버네티스, **자체호스팅**은 개인이나 회사 내 팀이 소유하거나 운영하는 시스템에 쿠버네티스를 직접 설치하고 구성하는 것을 의미.
2. Redis, Nginx ... 와 같은 소프트웨어를 설치하는 것과 동일
3. 자체 호스팅 쿠버네티스는 유연성과 권한을 최대로 보장
4. 실행할 쿠버네티스의 버전, 옵션 및 기능 설정, 클러스터 업그레이드 여부와 시기 등을 직접 결정할 수 있음

##### 자체 호스팅은 어렵다.
자체 호스팅은 인력, 기술, 엔지니어링 시간, 유지 보수, 문제 해결에 많은 자원을 요구
- 컨트롤 플레인이 고가용성을 보장 하는가
	- 마스터 노드가 다운 되거나 응답이 없는 경우에도 클러스터가 정상 작동하는가
	- 앱을 배포하거나 업데이트 할 수 있는가
	- 실행 중인 애플리케이션은 컨트롤 플레인 없이 fault-tolerant(장애 허용)을 유지하는가
- 워커 노드는 고가용성을 보장하는가
	- 여러 개의 워커 노드나 클라우드 가용성 영역에 문제가 발생할 경우 워크로드 실행이 중단되는가
	- 클러스터는 정상 작동하는가
	- 새로운 노드를 자동으로 프로비저닝하여 스스로 복구할 수 있는가 or 수동 조치 작업이 필요한가
- 클러스터는 안전하게 설정 되었는가
	- 내부 컴포넌트들은 TLS 암호화와 신뢰할 수 있는 인증서를 사용하여 통신하는가
	- 사용자와 애플리케이션은 클러스터 작업을 위한 최소한의 권한을 가지고 있는가
	- 컨테이너 보안 기본 값이 올바르게 설정 되었는가
	- 노드가 컨트롤 플레인 컴포넌트에 불필요한 접근을 하는가
	- 내부 etcd 데이터베이스에 대한 접근은 적절하게 제어되고 인증되는가
- 클러스터 내 모든 서비스는 안전한가
	- 인터넷에 접근할 수 있는 경우 제대로 인증되고 권한이 부여되는가
	- 클러스터 API에 대한 접근은 제한되어있는가
- 클러스터는 적합한가
	- 클라우드 네이티브 컴퓨팅 재단에서 정의한 쿠버네티스 클러스터 표준을 따르는가
- 클러스터 노드가 쎌 스크립트로 설치된 후 방치되지 않고 형상 관리되고 있는가
	- 각 노드의 운영체제와 커널은 업데이트 및 보안 패치 등의 작업이 필요
- 영구 스토리지를 포함하여 클러스터의 데이터가 제대로 백업되고 있는가
	- 복구 프로세스는 있는가
	- 복구 테스트를 얼마나 자주하는가
- 클러스터를 운영한다면 장기적으로 어떻게 관리할 것인가
	- 새로운 노드는 어떻게 프로비저닝하는가
	- 기존 노드의 구성 변경 사항을 롤아웃 하는가
	- 쿠버네티스 업데이트를 롤아웃하는가
	- 수요에 따라 노드 규모를 확장할 것인가
	- 정해둔 정책을 고수할 것인가

##### 자체 호스팅은 지속적인 관리가 필요

#### 쿠버네티스의 어려움
설치 및 관리가 간단하다고 알려져 있지만, 사실 **쿠버네티스는 꽤 어렵다.** 물론 쿠버네티스는 아주 간결하게 잘 설계돼있음.
허나 쿠버네티스가 하는 일이 너무나도 많음, 그리고 매 순간 복잡한 상황 처리를 하기 때문에 어려운 소프트웨어가 될 수 밖에 없음.

#### 관리 오버헤드
쿠버네티스 클러스터 전담 팀을 운영할 수 있는 큰 조직이라면 앞서 설명한 것은 큰 문제가 되지 않음.
하지만 중소 기업이나 소수의 엔지니어만 있는 스타트업이라면 자체 쿠버네티스 클러스터를 운영하는것은 부담

### 구입 또는 구축
#### 적게 실행하는 소프트웨어 (run less software) 철학
1. 표준 기술을 선택하라.
2. 차별화되지 않은 고된 일은 아웃소싱하라.
3. 지속적인 경쟁 우위를 창출하라.

# 쿠버네티스 오브젝트 다루기
쿠버네티스의 기본적인 오브젝트 **Pod**, **Deployment**, **Service**로 나눠서 설명
쿠버네티스 애플리케이션을 관리하는 핵심 도구인 헬름의 사용 방법도 확인

## 디플로이먼트 
프로그램 문제, 시스템 오류, 디스크 공간 부족 아니면 재해로 인해 컨테이너가 종료됐다면, 종료된 컨테이너가 상용 애플리케이션을 구동중이라면 누군가가 다시 터미널에 접속해 
```Shell
$ docker container run
```
이 명령어로 컨테이너를 다시 실행하기 전까지 사용자는 불편을 겪을 수 밖에 없음
관리 프로그램은 컨테이너를 감지하고  정지될 경우 즉시 실행해야함

전통적인 서버에서는 **systemd, runit, supervisord** 같은 도구를 사용
쿠버네티스에서는 **디플로이먼트**라는 관리자 기능을 사용

### 관리와 스케쥴링
쿠버네티스는 각 프로그램을 관리하기 위해 디플로이먼트 오브젝트 생성
디플로이먼트 오브젝트에는 해당 프로그램에 대한 정보(컨테이너 이미지 이름, 실행할 레플리카 수 등 컨테이너를 실행하기 위해 알아야하는 모든 정보)가 기록됨.

쿠버네티스 오브젝티브인 **컨트롤러**는 디플로이먼트 리소스를 관리함.
컨트롤러는 리소스가 존재하고 작동하는지 확인
만약 디플로이먼트 리소스가 특별한 이유 없이 레플리카 수를 채우지 못한 채 실행중이라면, 컨트롤러는 새로운 레플리카를 생성함. 만약 지정된 수를 초과하는 너무 많은 레플리카가 있는 경우 컨트롤러는 초과하는 레플리카를 종료.

실제로 디플로이먼트는 레플리카를 직접 관리하지 않음, 대신 **레플리카셋**(ReplicaSet)이라는 오브젝트를 자동으로 생성해서 처리하게 함.

### 컨테이너 재시작하기
디플로이먼트는 컨테이너가 작업을 마치고 종료되면, 이 컨테이너를 재시작함.
컨테이너에 크래시가 발생하거나, 시그널 이벤트나 kubectl로 컨테이너를 종료하는 경우에도 디플로이먼트는 재시작함.
컨테이너마다 재시작 정책을 다르게 설정할 수 있음.
재시작을 아예 하지 않거나, 정상적인 종료 방식이 아니라 비정상적으로 종료된 경우에만 재시작하게 할 수 있음.

기본값은 항상 재시작

### 디플로이먼트 조회하기
``` bash
$ kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
demo      1         1         1            1           21h
```
이런 방식으로 조회하고 출력됨.
자세하게 보고싶다면
```bash
$ kubectl describe deployments/demo
Name:                   demo
Namespace:              default
CreationTimestamp:      Tue, 08 May 2018 12:20:50 +0100
...
```
나머지 더 자세한 내용이 있지만 추후에 설명을 다시 써놓겠음

```Shell
Pod Template:
  Labels:  app=demo  
  Containers:
    demo:    
	  Image:        cloudnatived/demo:hello    
	  Port:         8888/TCP    
	  Host Port:    0/TCP    
	  Environment:  <none>    
	  Mounts:       <none>  
  Volumes:        <none>
```

## 파드
파드는 하나 이상의 컨테이너 그룹으로 구성된 쿠버네티스 오브젝트

디플로이먼트에서 개별 컨테이너를 직접 관리하지 않는 이유
- 컨테이너의 집합은 때때로 함께 스케쥴링되고, 동일 노드에서 실행되며, 로컬로 통신하거나 저장 공간을 공유해야하기 때문
- 예를 들어 블로그 애플리케이션은 깃 저장소, 컨텐츠를 동기화하는 컨테이너, 블로그 컨텐츠를 사용자에게 제공하는 Nginx 웹서버 컨테이너로 구성
	- 두 컨테이너는 데이터 공유해야하기 떄문에 파드 하나에 함께 스케쥴링 되어야함

파드 스펙에는 컨테이너 목록이 있음
```Shell
demo: 
  Image:  cloudnatived/demo:hello
  Port:8888/TCP
  Host Port: <0/TCP 
  Environment:  <none> 
  Mounts:       <none>
```

## 레플리카셋
디플로이먼트는 파드를 직접 관리하지 않음. 파드 관리는 레플리카셋 오브젝트가 수행함.

레플리카셋은 동일한 파드 집합이나 **레플리카**들을 관리
스펙보다 너무 적거나 많은 파드가 존재하면, 레플리카셋 컨트롤러는 상태를 바로잡기 위해 일부 파드를 실행하거나 중지함

디플로이먼트는 레플리카셋을 관리하며 레플리카들이 애플리케이션의 새로운 버전을 롤아웃하여 업데이트할 때의 작동을 제어
디플로이먼트를 업데이트하면 새 파드를 관리하기 위한 레플리카셋이 생성됨.
업데이트가 완료되면 이전 레플리카셋과 파드는 종료
<img src="/assets/Pasted%20image%2020221021105153.png" />
![](Pasted%20image%2020221021105153.png)
## 의도한 상태 유지하기
쿠버네티스 컨트롤러는 각 리소스에서 지정한 '의도된 상태'를 클러스터의 실제 상태와 지속적으로 비교하고 동기화하기 위해 필요한 작업 수행함.

의도된 상태와 실제 상태를 일치시키기 위한 조정 작업이 영원히 반복되므로 이 과정을 **조정 루프(reconciliation loop)** 라고 함.
```shell
$ kubectl delete pods --selector app=demo
pod "demo-54df94b7b7-qgtc6" deleted
```
이렇게 파드를 중지하고 재 조회를 해보면
```shell
$ kubectl get pods --selector app=demo 
NAME                    READY     STATUS        RESTARTS   AGE
demo-54df94b7b7-hrspp   1/1       Running       0          5s
demo-54df94b7b7-qgtc6   0/1       Terminating   1          22h
```
이렇게 pod가 죽음과 동시에 다시 재실행됨.
하지만 디플로이먼트를 종료한 상태에서 내리면 모든 파드가 정리됨.
```shell
$ kubectl delete all --selector app=demo
pod "demo-54df94b7b7-hrspp" deleted
service "demo" deleted
deployment.apps "demo" deleted
```

## 쿠버네티스 스케줄러
'디플로이먼트가 파드를 생성하고 쿠버네티스는 요정된 파드를 실행한다.'
쿠버네티스 스케줄러는 이 과정을 책임지는 컴포넌트, 디플로이먼트가 관련된 레플리카셋을 통해 새로운 레플리카가 필요하다고 결정하면 쿠버네티스 데이터베이스에 파드 리소스를 직접 생성,
동시에 이 파드는 스케줄러 수신함과 같은 대기열에 추가

파드가 노드에 스케줄링 되면 노드에서 실행중인 kubelet이 실제로 컨테이너를 실행
#쿠버네티스스케줄러의작동원리 #줄리아에번스

## YAML 형식의 리소스 manifest
kubectl run 명령어를 사용해 디플로이먼트를 생성하는 것은 제한적임
이미지 이름이나 버전과 같은 디플로이먼트 스펙을 변경한다고 생각하면,
kubectl delete 명령어를 사용해 기존 디플로이먼트를 삭제하고 값이 수정된 새로운 디플로이먼트를 생성하게 됨, => 번거로움

쿠버네티스는 근본적으로 **선언형** 시스템, 실제 상태와 의도한 상태를 계속해서 조정해 맞춰감.
의도한 상태에서 디플로이먼트 스펙만 수정하면 쿠버네티스가 나머지 일을 모두 처리

### 리소스는 데이터
디플로이먼트, 파드와 같은 쿠버네티스 리소스는 모두 내부 데이터베이스에 기록
조정 루프는 데이터베이스 내 기록의 변경 사항을 감시하고 적절하게 동작

### Deployment Manifest
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
  metadata:  
    name: demo  
    labels:    
      app: demo
  spec:  
    replicas: 1  
    selector:    
      matchLabels:      
        app: demo  
    template:    
      metadata:      
        labels:        
          app: demo    
      spec:      
        containers:        
          - name: demo          
            image: cloudnatived/demo:hello
            ports:
		    - containerPort: 8888
```
예시를 활용해 작성

### kubectl apply
YAML Manifest를 클러스터에 직접 전달하는 방법 
- kubectl apply
```shell
$ cd hello-k8s
$ kubectl apply -f k8s/deployment.yaml 
deployment.apps "demo" create
```

```shell
$ kubectl get pods --selector app=demo 
NAME                    READY     STATUS    RESTARTS   AGE
demo-6d99bf474d-z9zv6   1/1       Running   0          2m
```
파드를 웹 브라우저에서 접속 가능하게 하려면 배포한 파드에 연결할 수 있게 해주는 쿠버네티스 서비스 리소스를 생성해야함.

### 서비스 리소스
데모 애플리케이션 예제처럼 네트워크 연결이 필요한 파드가 있다면?
파드의 IP 주소를 찾고 애플리케이션의 포트 번호를 연결하면 됨.
하지만 IP 주소는 파드가 재시작할 때마다 변경될 수 있기 때문에 계속해서 상태를 확인해야함
서비스 리소스는 자동으로 라우팅되는 영구적인 IP 주소와 DNS 주소를 제공
고급 라우팅과 TLS 인증서를 사용할 수 있는 인그레스 리소스가 필요함

서비스는 웹 프록시나 로드 밸런서와 같이 요청을 백엔드 파드의 세트로 전달
또한 웹 포트뿐만 아니라 스펙의 포트 부분에 명시된 모든 포트로 트래픽을 전달 가능
<img src="/assets/Pasted%20image%2020221021144406.png" />
![](Pasted%20image%2020221021144406.png)
```yaml
apiVersion: v1
kind: Service
  metadata:  
    name: demo
  labels:    
    app: demo
spec:  
  ports:  
  - port: 9999    
    protocol: TCP    
    targetPort: 8888  
  selector:    
    app: demo  
  type: ClusterIP
```
이전 디플로이먼트 리소스와 비슷해보임, 그러나 kind 항목이 deployment가 아닌 service이며 spec은 단순하게 port, selector, type만 포함
서비스가 9999포트를 파드의 8888포트로 포워딩하고 있음

- selector는 서비스에 라우팅할 파드를 알려주는 역할.
	- 요청은 지정된 레이블 집합과 일치하는 모든 파드로 전달
	- 무작위 전달 (Load Balance)

```shell
$ kubectl apply -f k8s/service.yaml
service "demo" created

kubectl port-forward service/demo 9999:8888
Forwarding from 127.0.0.1:9999 -> 8888
Forwarding from [::1]:9999 -> 8888
```

### kubectl로 클러스터 조회
kubectl 도구는 쿠버네티스의 중요한 도구 중 하나, 설정을 적용하고 리소스를 생성, 수정, 제거할 수 있으며 클러스터에 존재하는 리소스 상태에 대한 정보를 조회 가능

```shell
$ kubectl get nodes
NAME                 STATUS    ROLES     AGE       VERSION
docker-for-desktop   Ready     master    1d        v1.10.0
```

모든 타입의 리소스를 확인하고 싶으면 kubectl get all 명령어 사용
포괄적인 정보는 **kubectl description**
- kubectl description은 매우 중요한 명령어
	- 실제로 트러블 슈팅에서 pod 혹은 deployment가 왜 안도는지,
	- node가 왜 안올라가는지
	- 여러 문제를 확인할 수 있도록 하는 도구

```bash
$ kubectl describe pod/demo-dev-6c96484c48-69vss
Name:           demo-dev-6c96484c48-69vss
Namespace:      default
Node:           docker-for-desktop/10.0.2.15
Start Time:     Wed, 06 Jun 2018 10:48:50 +0100
...
Containers:  
  demo:    
    Container ID:   docker://646aaf7c4baf6d...
    Image:          cloudnatived/demo:hello...
Conditions:  
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
...
Events:  
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  1d    default-scheduler  Successfully assigned demo-dev...
  Normal  Pulling    1d    kubelet            pulling image "cloudnatived/demo...
...
```
예제 출력 결과에서 kubectl이 이미지 식별자, 상태와 같은 컨테이너 정보와 컨테이너에서 발생한 이벤트 목록을 제공

### 리소스 활용하기
YAML 메니페스트를 통해 배포하지만, 겹치는 항목이 많다는것이 흠.
한번만 지정하고 쿠버네티스 메니페스트를 통해 참조할 수 있도록 반복되는 항목의 YAML 파일을 만들고 관리할 수있도록 **헬름**을 사용

## 헬름: 쿠버네티스 패키지 매니저
헬름은 쿠버네티스 패키지 매니저, helm 명령줄 도구를 사용(혹은 직접 만들거나)하면 애플리케이션을 설치하고 설정 가능, 헬름 **차트**라 불리는 패키지를 생성하면 애플리케이션을 실행하는데 필요한 리소스, 의존성, 구성 가능한 변수를 지정 가능

### 헬름 설치
운영체제 별 설치가이드 (https://helm.sh/docs/using_helm/#installing-helm)

설치한 후에는 클러스터에 접근할 수 있도록 권한을 부여하는 쿠버네티스 리소스를 만들어야함.
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

#### 헬름 ServiceAccount 중요 설명
이 리소스는 tiller라는 접근 권한을 가진 서비스계정을 만드는 작업임
그 계정에 ClusterRoleBinding, 즉 클러스터 접근 역할을 부여해주는 것으로 k8s 그룹 전체, kube-system의 전체 권한을 얻는 방법

해당 리소스를 통해서 적용 후 helm을 해당 service account에 등록해주도록 해야함.
```bash
$ cd ../hello-helm
kubectl apply -f helm-auth.yaml
serviceaccount "tiller" created
clusterrolebinding.rbac.authorization.k8s.io "tiller" created
```

필요한 권한을 tiller라는 이름의 service account로 얻었으니 해당 권한을 부여해주기
``` bash
$ helm init --service-account tiller
$HELM_HOME has been configured at /Users/john/.helm.
```
헬름이 초기화 완료하기 위해선 5분 정도의 시간 필요
```bash
$ helm version
Client: &version.Version{SemVer:"v2.9.1",
GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.9.1",
GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
```

### 헬름 차트 설치
총 네 개의 yaml 파일과 templates

```yaml
# Chart.yaml
name: demo
sources:
- https://github.com/cloudnativedevops/cloudnatived
version: 1.0.1
```

```yaml
# production_values.yaml
environment: production
```

```yaml
# staging-values.yaml
environment: staging
```

```yaml
# values.yaml
environment: development
container:
  name: demo
  port: 8888
  image: cloudnatived/demo
  tag: hello
replicas: 1
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.container.name }}
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.container.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.container.name }}
        environment: {{ .Values.environment }}
    spec:
      containers:
        - name: {{ .Values.container.name }}
          image: {{ .Values.container.image }}:{{ .Values.container.tag }}
          ports:
            - containerPort: {{ .Values.container.port }}
          env:
            - name: environment
              value: {{ .Values.environment }}
```

```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.container.name }}-service
  labels:
    app: {{ .Values.container.name }}
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: {{ .Values.container.port }}
  selector:
    app: {{ .Values.container.name }}
  type: LoadBalancer
```

헬름 차트에서 사용할 파일들, 각각의 환경변수를 사용할 수 있도록 환경변수 지정과 사용

먼저 헬름을 사용해서 애플리케이션을 설치한다면, 이전에 작업했던 모든 디플로이먼트 리소스를 정리해야함.
```bash
$ kubectl delete all --selector app=demo
```

```bash
$ helm install --name demo ./k8s/demo
NAME:   demo
LAST DEPLOYED: Wed Jun  6 10:48:50 2018
NAMESPACE: default
STATUS: DEPLOYED
RESOURCES:
==> v1/Service
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
demo-service    ClusterIP  10.98.231.112  <none>       80/TCP    0s
==> v1/Deployment
NAME      DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
demo      1        1        1           0          0s
==> v1/Pod(related)
NAME                       READY  STATUS             RESTARTS  AGE
demo-6c96484c48-69vss      0/1    ContainerCreating  0         0s
```
이전 예제와 같이 헬름이 (파드를 시작한) 디플로이먼트 리소스와 서비스를 생성했음을 확인할 수 있음. helm install은 **헬름 릴리스**라는 쿠버네티스 오브젝트를 생성하여 이를 수행

### 차트, 리포지터리, 릴리스
헬름의 중요한 용어 세 가지
- **차트**, 쿠버네티스에서 애플리케이션을 실행하는데 필요한 모든 리소스 정의를 포함한 헬름 패키지
- **리포지터리**, 차트가 모여 있는 공유할 수 있는 공간
- **릴리스**, 쿠버네티스 클러스터에서 실행되는 차트의 특정 인스턴스
서로 다른 사이트를 서비스하는 Nginx 웹서버 차트와 같이 하나의 차트를 동일한 클러스터에 여러 번 설치하고 실행할 수 있음, 차트의 인스턴스는 서로 다른 릴리스

### 헬름 릴리스 목록 확인

```bash
$ helm list
NAME    REVISION UPDATED                  STATUS   CHART      NAMESPACE
demo    1        Wed Jun  6 10:48:50 2018 DEPLOYED demo-1.0.1 default
```
특정 릴리스의 정확한 상태를 확인하려면 helm status와 릴리스 이름을 입력하고 실행


## 쿠버네티스 오브젝트 다루기 정리
- 파드는 쿠버네티스의 기본 작업 단위로 스케줄링이 될 단일 컨테이너나 컨테이너의 그룹을 지정한다. 
- 디플로이먼트는 파드를 선언적으로 관리하는 고수준 쿠버네티스 리소스다. 배포, 스케줄링, 업데이트 작업을 수행하며 필요하다면 파드를 재시작한다. 
- 서비스는 쿠버네티스의 로드 밸런서나 프록시에 해당하며 IP 주소나 DNS 이름을 통해 트래픽을 일치하는 파드로 라우팅한다.  
- 쿠버네티스 스케줄러는 노드에서 아직 실행되지 않는 파드를 감시하고 적합한 노드를 찾은 다음 해당 노드의 kubelet이 파드를 실행하도록 지시한다.   
- 디플로이먼트와 같은 리소스는 쿠버네티스의 내부 데이터베이스에 레코드로 표시된다. 외부적으로 이러한 리소스는 YAML 형식의 텍스트 파일(매니페스트라고 함 )로 표현된다. 
- 쿠버네티스를 활용한 클라우드 네이티브 데브옵스매니페스트는 리소스의 의도한 상태를 선언한다.  
- kubectl은 쿠버네티스와 상호작용하기 위한 주요 도구로 매니페스트를 적용하고 리소스를 조회, 변경, 삭제하는 등의 많은 작업을 수행할 수 있다.  
- 헬름은 쿠버네티스 패키지 매니저로 쿠버네티스 애플리케이션을 쉽게 구성하고 배포할 수 있게 해준다. 직접 원시 YAML 파일을 관리할 필요 없이 하나의 값 세트(애플리케이션 이름이나 수신 포트와 같은 )나 템플릿의 세트를 사용하여 쿠버네티스 YAML 파일을 생성할 수 있게 해준다.

