Master Node의 클러스터를 생성하기 위한 과정 기술
# 구성 방식

On Promise 방식의 서버 2대 구성, 마스터노드로 생각하는 서버와 worker는 분리돼있음
- 마스터 노드
- 워커 노드1
	-  차 후에 워커 노드를 추가 생산하는 로드맵을 가짐

쿠버네티스 클러스터를 쉽게 구축하고 관리할 수 있는 **kubeadm** 도구를 통해 클러스터 설치

# 사전작업
마스터 노드와 워커 노드가 될 서버들에 해줘야할 작업들
1. [[Network Time Protocol (NTP)]] 서버 동기화 
2. 스왑메모리 해제
3. 도커 엔진 설치
4. 도커 데몬 systemd 로 변경

## NTP 서버 동기화
```shell
$ sudo yum install ntp
```

## Master Node
마스터 노드에서는 설치 후 서비스 리로드, 서비스 동작확인만 해주기로 함. 추후에 prometheus / grafana를 통한 통계 화면 설정

```shell
$ systemctl start ntpd
$ sudo service ntpq reload
$ sudo ntpq -p
```

**pool**로 설정돼있는 ntp 서버들에서 시간을 받아오는 결과를 확인할 수 있음

## Workder Node
```shell
$ sudo yum install ntp
$ sudo vi /etc/ntp.config

# 주석이 해제되어 있는 모든 pool과 server를 주석처리하고 마스터노드 IP 추가
server 172.1.1.0

$ sudo systemctl restart ntp
$ sudo ntpq -p
```
설치 후 ntp 설정을 변경하고 서비스 재시작 서비스 동작 확인을 거쳐야함

## 스왑메모리 해제(swapoff)
쿠버네티스 클러스터는 메모리 스왑이 활성화 돼있는 것을 허용하지 않음.

	메모리 스왑이 활성화 돼 있으면 컨테이너의 설능이 일관되지 않을 수 있기 때문에,
	대부분의 쿠버네티스 설치 도구는 메모리 스왑을 허용하지 않음.

kubeadm 역시 스왑메모리를 허용하지 않음.
```shell
$ swapoff -a
```
추가적으로 혹시 노드 들의 재부팅이 잦을 예정이라면 fstab에서  swap에 대한 부분을 주석처리하는 것도 좋은 방법

```shell
$ sudo vi /etc/fstab

>
# swap was on /dev/sda2 during installation
# UUID=e09948a3-(중략)...-79d1d6 none            swap    sw              0       0
```

## SELINEX 비활성화
```shell
$ setenforce 0
$ sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```

## 방화벽 해제
```shell
$ systemctl disable firewalld
$ systemctl stop firewalld
```

## 도커 엔진 설치

```shell
$ yum install -y yum-utils
```

도커 엔진을 설치할 수 있도록 저장소 추가
```shell
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

도커 엔진 최신버전 설치
```shell
$ yum install docker-ce docker-ce-cli containerd.io -y
```

도커 엔진 실행
```shell
$ systemctl start docker
$ systemctl enable docker
$ systemctl status docker
```

## 도커 데몬 변경
도커 데몬의 컨테이너 런타임을 변경해야함. [컨테이너 런타임](Docker.md#컨테이너%20런타임) 문서 확인.
일반적으로 리눅스에서 프로세스를 관리하며 리소스 컨트롤하는 프로세스는 systemd와 cgroubfs로 나뉨. 컨테이너의 경우 cgroupfs를 사용하는데 일반적 프로세스인 systemd와 충돌을 일으켜 효율적인 자원관리가 안될 수 있음. 그래서 systemd로 컨테이너의 런타임을 교체해줌.

```shell
$ sudo mkdir /etc/docker
$ cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",

  "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
  "default-runtime": "nvidia"
}
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

내용 중 runtimes와 default-runtime은 쿠버네티스에서 GPU 자원을 쓰기 위해 추가하는 부분, GPU가 없다면 지워도 됨.

컨테이너 런타임의 변경으로 인한 설치과정이 추가됨.
아래는 docker가 설치돼있고 cri-dockerd를 설치한다는 가정 하에 진행
```shell
# Run these commands as root
# home 위치에
git clone https://github.com/Mirantis/cri-dockerd.git
###Install GO###
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile

cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```

## IPTABLES 설정
- br_netfilter 모듈이 로드 되었는지 확인
- sysctl.conf 에서 net.bridge.bridge-nf-call-iptables 값을 1로 설정
```shell
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf br_netfilter EOF
```
```shell
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf net.bridge.bridge-nf-call-ip6tables = 1 net.bridge.bridge-nf-call-iptables = 1 EOF 
$ sudo sysctl --system
```

# 저장소 추가 및 kubeadm 설치
gpg 저장소를 추가하여 kubeadm을 받을 수 있는 환경을 만들고 업데이트를 실행해줌. root 계정에서 명령어를 사용한다면 sudo 생략 가능.
아래 작업은 마스터와 워커 노드 각각 공통적으로 적용.
```shell
#centos
# /etc/yum.repos.d/kubernetes.repo
$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo 
[kubernetes] name=Kubernetes baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1 
gpgcheck=1 
# 이 부분은 원래 1로 해서 위변조 체크를 해야함, ncp에서는 외부 보안 연결 때문에 인증불가
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg 
exclude=kubelet kubeadm kubectl EOF
```
```shell
$ yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
$ systemctl enable --now kubelet
```
```shell
#Ubuntu
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ sudo cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main 
EOF
$ sudo yum update
```

## 쿠버네티스 버전 고정
centos에서는 exclude에 넣은 패키지는 업데이트 되지 않음.

```shell
#ubuntu
$ sudo apt-mark hold kubelet kubeadm kubectl
```

# 클러스터 실행
## Master Node
마스터노드에서는 kubeadm의 init 명령어를 이용해 쿠버네티스 클러스터를 시작할 수 있음.
아래의 명령어 중 pod-network-cidr 옵션에 들어가 있는 IP는 기억해야함
	네트워크 설정 시 사용
	

```shell
$ sudo kubeadm init --apiserver-advertise-address ${마스터 노드IP} --pod-network-cidr=${설정할 worker들의 IP범주 예시:192.168.10.0/24}
```

```shell
[init] Using Kubernetes version: v1.22.3
[preflight] Running pre-flight checks 
[preflight] Pulling images required for setting up a Kubernetes cluster 
[preflight] This might take a minute or two, depending on the speed of your internet connection 
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull' 
[certs] Using certificateDir folder "/etc/kubernetes/pki" 
[certs] Generating "ca" certificate and key 
[certs] Generating "apiserver" certificate and key 
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.0.101] 
[certs] Generating "apiserver-kubelet-client" certificate and key 
[certs] Generating "front-proxy-ca" certificate and key 
[certs] Generating "front-proxy-client" certificate and key 
[certs] Generating "etcd/ca" certificate and key 
[certs] Generating "etcd/server" certificate and key 
[certs] etcd/server serving cert is signed for DNS names 
[k8s-master localhost] and IPs [192.168.0.101 127.0.0.1 ::1] 
[certs] Generating "etcd/peer" certificate and key 
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [192.168.0.101 127.0.0.1 ::1] 
[certs] Generating "etcd/healthcheck-client" certificate and key 
[certs] Generating "apiserver-etcd-client" certificate and key 
[certs] Generating "sa" key and public key [kubeconfig] Using kubeconfig folder "/etc/kubernetes" 
[kubeconfig] Writing "admin.conf" kubeconfig file 
[kubeconfig] Writing "kubelet.conf" kubeconfig file 
[kubeconfig] Writing "controller-manager.conf" kubeconfig file 
[kubeconfig] Writing "scheduler.conf" kubeconfig file 
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env" 
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml" 
[kubelet-start] Starting the kubelet 
[control-plane] Using manifest folder "/etc/kubernetes/manifests" 
[control-plane] Creating static Pod manifest for "kube-apiserver" 
[control-plane] Creating static Pod manifest for "kube-controller-manager" 
[control-plane] Creating static Pod manifest for "kube-scheduler" 
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests" 
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s 
[apiclient] All control plane components are healthy after 12.505516 seconds 
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace 
[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster 
[upload-certs] Skipping phase. Please see --upload-certs 
[mark-control-plane] Marking the node k8s-master as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers] 
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule] 
[bootstrap-token] Using token: 7md2wt.1h1xfuzk3xgn0lft 
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles 
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes 
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials 
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token 
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster 
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace 
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key 
[addons] Applied essential addon: CoreDNS 
[addons] Applied essential addon: kube-proxy
```

```shell
Your Kubernetes control-plane has initialized successfully! 

To start using your cluster, you need to run the following as a regular user: 

 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
 sudo chown $(id -u):$(id -g) $HOME/.kube/config 
 
Alternatively, if you are the root user, you can run: 
 export KUBECONFIG=/etc/kubernetes/admin.conf 

You should now deploy a pod network to the cluster. 
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
 https://kubernetes.io/docs/concepts/cluster-administration/addons/
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.101:6443 --token 7md2wt.1h1xfuzk3xgn0lft \ --discovery-token-ca-cert-hash sha256:709dd217e4c3b6698017a255f64e56e1d0e41bdb4090e27cf23265e06e341a95
```

위 명령어의 결과 2가지
1. 마스터노드에서 이어서 작업해줘야하는 부분
2. 워커노드들에서 실행할 join 명령어

```shell
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
이 내용이 조금 다르게 나오더라도 상관 없음. 출력되는 부분 복사해서 마스터 노드에서 붙여넣기로 실행, 아래 더 출력된 init 명령어는 워커노드에서 실행

## Worker Node
워커 노드에서 터미널을 열고 마스터 노드에서 init을 해서 나온 join 명령어를 실행
```shell
$ sudo kubeadm join ${마스터 노드 IP}:6443 - token caamm9.q00z842...(생략) \ 
      --discovery-token-ca-cert-hash sha256:e12dcc40156...(생략)
```

## 임시 확인
네트워크 연결까지 해야 클러스터 구성이 제대로 됐는지 확인할 수 있지만 일단 init과 join이 잘 돼있는지 다음과 같이 확인 가능, 마스터 노드에서 명령어 실행
```shell
$ kubectl get nodes
Name        STATUS      ROLES                   AGE      VERSION
master      NotReady    control-plane,master    99s      v1.22.0
worker1     NotReady    <none>                  42s      v1.22.0
worker2     NotReady    <none>                  46s      v1.22.0
woeker3     NotReady    <none>                  48s      v1.22.0
```

# 네트워크 실행

## Calico
Calico 오버레이 네트워크를 설정, 설정 파일을 내려받아 클러스터가 구축된 상태에서 kubectl apply -f로 실행하여 네트워크를 적용하면 워커노드의 상태를 실시간 네트워크 동신을 통해 알 수 있게 됨.

여기서 아까 마스터 노드에 클러스터를 init할 때 입력한 IP 범주가 필요함.
```shell
$ wget https://docs.projectcalico.org/manifests/calico.yaml
$ sed -i -e 's?192.168.0.0/16?${우리가 설정한 IP 범주}/24?g' calico.yaml
$ kubectl apply -f calico.yaml
```
네트워크가 잘 생성 및 적용됐는지 확인해 보려면 생성된 파드를 확인
```shell
$ kubectl get pods --namespace kube-system
```

# 작동확인

## 클러스터 구축 확인
클러스터의 구성 자체를 확인하려면 다음 명령어를 치고 status 확인
```shell
$ kubectl get nodes
```
위 명령어를 실행했을 때 나오는 출력 결과는 마스터 노드와 워커노드의 현재 상태임. 모든 워커 노드가 Ready를 만족 하면 클러스터에 문제가 없음.
