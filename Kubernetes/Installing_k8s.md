# Kubernetes 설치

목차
- 개요
- 구성정보
  - Container Platform 구성 정보
  - H/W, S/W 구축 및 구성
- OS 구성
- Container runtimes 설치(Docker)
- Kubernetes 설치
  - Installing kubeadm
  - Kubernetes 설치 및 구성
  - Dashboard 설치
  - Resource monitoring 도구 설치
- Pesistent Volume(Storage) 구성
  - NFS

**이 문서는 2020년 4월 기준으로 작성되어 있어 인터넷 및 Book에 나온 내용하고 차이가 있을 수 있음**

---
## 개요

오케스트레이션 Tool중에 하나인 Kubernetes는 분산 환경에서 컨테이너를 통합 관리를 해준다.<br>
하드웨어 Infrastructure를 추상화 하여 데이터센터 전체를 하나의 방대한 리소스로 간주하여 개발자는 실제 서버를 의식할 필요 없이 컨테이너 어플리케이션을 Deploy하여 실행할 수 있다.

- Container Platform 환경 구성 Tool List

| OS | Container runtimes | Orchestration | CNI(Network) |
| -- | -- | -- | -- |
| **CentOS** | **Docker** | **Kubernetes** | **Calico** |
| RHEL | CRI-O | Apache Mesos(DC/OS) | Weave Net |
| Ubuntu | Containerd | Docker Swarm | Flannel |
| Container Linux(CoreOS) | rkt | |
| RHCOS | | | |
| Windows | | |
| Orcle Linux | | |
| Suse Linux | | |

> * Container Platform 상용 제품 리스트 및 구성 정보<br>
>> Openshift(Redhat) : Red Hat Enterprise Linux CoreOS (RHCOS) 또는 RHEL + CRI-O + K8s  
>> Cloud Pak(IBM) : Openshift + Cloud Pak  
>> HPE Container Platform(HPE) : ?? + K8s  
>> Pivotal Cloud Foundry(DellEMC) : ?? + K8s

- Installing Kubernetes with deployment tools 리스트

| k8s Installation Tool |
| :--: |
| **kubeadm** |
| kops |
| KRIB |
| Kubespray |

## 구성정보

- Network 구성
  - DNS : 8.8.8.8(google public dns), 1.1.1.1(cloudflare public dns), 168.126.63.1(kt public dns)

| Hostname | Network Address | Subnet | Netmask/CIDR | IP Address(range) | G/W | Broadcast |
| -- | -- | -- | -- | -- | -- | -- |
| NAT Network<br> (NAT-Network IP) | 172.16.2.0 | 172.16.2.0 | 255.255.255.0/24 | 172.16.2.1 ~ 172.16.2.254 | 172.16.2.1 | 172.16.2.255 |
| 내부네트워크(intnet IP) | 10.0.0.0 | 10.0.0.0 | 255.255.0.0/16 | 10.0.0.1 ~ 10.0.255.254 | X | 10.0.255.255
| 호스트전용어댑터 | 192.168.56.0 | 192.168.56.0 | 255.255.255.0/24 | 192.168.56.1 ~ 192.168.56.254 | X | 192.168.56.255

- Hardware 구성

| Hostname | NAT-Network IP | intnet IP | 호스트전용어댑터 | CPU | MEM | DISK |
| -- | -- | -- | -- | -- | -- | -- |
| k8s-master1 | 172.16.2.101 | 10.0.0.101 | 192.168.56.101 | **2** | 2G | 8G |
| k8s-workernode1 | 172.16.2.111 | 10.0.0.111 | 192.168.56.111 | 1 | 2G | 8G |
| k8s-workernode2 | 172.16.2.112 | 10.0.0.112 | 192.168.56.112 | 1 | 2G | 8G |
| k8s-workernode3 | 172.16.2.113 | 10.0.0.113 | 192.168.56.113 | 1 | 2G | 8G |
| k8s-nfs1 | 172.16.2.201 | 10.0.0.201 | 192.168.56.201 | 1 | 2G | 8G |

> nfs1서버는 IaC(ansible)기능도 포함한다.

- Software 구성

| S/W | Product | Version | 
| -- | -- | -- |
| OS | CentOS | 7.7-1908 | 
| Container Runtime | Docker | 19.03.8-ce | 
| Orchestration tool | Kubernetes | v1.17 | 
| IaC tool | ansible | 2.4.2.0-2.el7 |
| VCS | git | 1.8.3.1-21.el7_7 |
| CI/CD tool | Jenkins | 추후 설치 예정 |

> VCS는 local git(pc or 노트북) 과 remote git(gitlab 또는 github) 연동  
> remote git은 Jenkins 와 연동

- Overlay Network 환경

| Container(Docker) IP | Pod IP | Service IP(Cluster IP) | Node IP(Host) | Port range |
| -- | -- | -- | -- | -- |
| 172.17.0.0/16 | 10.10.0.0/16 | 10.96.0.0/12 (default) | 172.16.2.111~113<br>172.16.2.101 | 30000 ~ 32767 |

---

## OS 구성

대상서버 : k8s-master1, k8s-workernode1~3

- Firewalld 중지

```
# Docker는 iptables을 이용하기 때문에 firewalld 중지
systemctl stop firewalld
systemctl disable firewalld
```

- Selinux 중지

```
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```

- Hosts 추가

```
echo "

# k8s nodes 
# $(date)
172.16.2.101  k8s-master1
172.16.2.111  k8s-workernode1
172.16.2.112  k8s-workernode2
172.16.2.113  k8s-workernode3
10.0.0.201    k8s-nfs1" >> /etc/hosts
```

- Hosts 변경
```
hostnamectl set-hostname --static 호스트네임
```

- Swap 중지

```
swapoff -a && sed -e '/swap/ s/^/#/' -i /etc/fstab
```

- Letting iptables see bridged traffic

```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

- nfs-client 설치

```
yum install nfs-utils
```

- Reboot

```bash
shutdown -r now
```

## Container runtimes 설치

```bash
curl -o /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
yum install git
yum install docker-ce-19.03.4 docker-ce-cli-19.03.4 containerd.io-1.2.10
systemctl enable docker
systemctl start docker
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
mkdir -p /etc/systemd/system/docker.service.d
systemctl daemon-reload
systemctl restart docker
```

## Kubernetes 설치
### Installing kubeadm

#### 대상서버 : k8s-master1(control-plane node), 

- k8s repository
```
cat <<'EOF' > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

- kubeadm version list 확인

```
yum list kubeadm kubelet kubectl --showduplicates --disableexcludes=kubernetes | sort --version-sort -k2 | tail -20
또는
yum list kubeadm kubelet kubectl --showduplicates --disableexcludes=kubernetes | sort -n -t. -k3 | tail -20
```

- [Installing kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl "installing packages list")

```
yum install kubelet-1.17.4 kubeadm-1.17.4 kubectl-1.17.4 --disableexcludes=kubernetes
systemctl enable --now kubelet
```
### Kubernetes 설치 및 구성

#### 대상서버 : k8s-master1(control-plane node)

- kubeadm init 전 사전 TEST (--dry-run)

```
kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=172.16.2.101 --dry-run
```

- kubeadm init

```
kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=172.16.2.101
```

출력로그

```
[root@k8s-master1 ~]# kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=172.16.2.101
I0407 17:56:13.074298    2991 version.go:251] remote version is much newer: v1.18.0; falling back to: stable-1.17
W0407 17:56:13.770846    2991 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0407 17:56:13.770894    2991 validation.go:28] Cannot validate kubelet config - no validator is available
[init] Using Kubernetes version: v1.17.4
[preflight] Running pre-flight checks

...생략

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.2.101:6443 --token ty27yj.vvgzmn7ifenfmh6f \
    --discovery-token-ca-cert-hash sha256:49c44083ec868ba9a5481652b2887cade71f6cfd9a4c44a8586995fab2b8cd1e
```

- kubectl 명령어와 k8s Cluster(API) 통신을 위해 아래 command 실행

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get po --all-namespaces -o wide
```

- kubectl 명령어 완성(tab키)

```
yum install bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
kubectl completion bash >/etc/bash_completion.d/kubectl
kubectl [tab 2번]
```

출력로그

```
[root@k8s-master1 ~]# kubectl get po --all-namespaces -o wide
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
kube-system   coredns-6955765f44-fcvwn              0/1     Pending   0          80m   <none>         <none>        <none>           <none>
kube-system   coredns-6955765f44-pjn89              0/1     Pending   0          80m   <none>         <none>        <none>           <none>
kube-system   etcd-k8s-master1                      1/1     Running   0          80m   172.16.2.101   k8s-master1   <none>           <none>
kube-system   kube-apiserver-k8s-master1            1/1     Running   0          80m   172.16.2.101   k8s-master1   <none>           <none>
kube-system   kube-controller-manager-k8s-master1   1/1     Running   0          80m   172.16.2.101   k8s-master1   <none>           <none>
kube-system   kube-proxy-8m7dh                      1/1     Running   0          80m   172.16.2.101   k8s-master1   <none>           <none>
kube-system   kube-scheduler-k8s-master1            1/1     Running   0          80m   172.16.2.101   k8s-master1   <none>           <none>
```

- Calico 설치

```
mkdir $HOME/k8s_src && cd $HOME/k8s_src
curl https://docs.projectcalico.org/manifests/calico.yaml -O
```

- Calico 설정
  - 주석제거 및 CALICO_IPV4POOL_CIDR 대역 변경
  - 띠어 쓰기 주의(yaml형식)

```bash
vi calico.yaml
===============
- name: CALICO_IPV4POOL_CIDR
  value: "10.10.0.0/16"
```
```
kubectl apply -f calico.yaml
```

출력로그

```
[root@k8s-master1 k8s_src]# kubectl apply -f calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
```

출력로그

```
[root@k8s-master1 k8s_src]# kubectl get po --all-namespaces -o wide
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE    IP              NODE          NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-5554fcdcf9-bccf8   1/1     Running   0          3m6s   10.10.159.130   k8s-master1   <none>           <none>
kube-system   calico-node-knxwq                          1/1     Running   0          3m6s   172.16.2.101    k8s-master1   <none>           <none>
kube-system   coredns-6955765f44-fcvwn                   1/1     Running   0          159m   10.10.159.129   k8s-master1   <none>           <none>
kube-system   coredns-6955765f44-pjn89                   1/1     Running   0          159m   10.10.159.131   k8s-master1   <none>           <none>
kube-system   etcd-k8s-master1                           1/1     Running   0          159m   172.16.2.101    k8s-master1   <none>           <none>
kube-system   kube-apiserver-k8s-master1                 1/1     Running   0          159m   172.16.2.101    k8s-master1   <none>           <none>
kube-system   kube-controller-manager-k8s-master1        1/1     Running   0          159m   172.16.2.101    k8s-master1   <none>           <none>
kube-system   kube-proxy-8m7dh                           1/1     Running   0          159m   172.16.2.101    k8s-master1   <none>           <none>
kube-system   kube-scheduler-k8s-master1                 1/1     Running   0          159m   172.16.2.101    k8s-master1   <none>           <none>
```

#### 대상서버 : k8s-workernode1~3

- Worker nodes join

```
kubeadm join 172.16.2.101:6443 --token ty27yj.vvgzmn7ifenfmh6f \
    --discovery-token-ca-cert-hash sha256:49c44083ec868ba9a5481652b2887cade71f6cfd9a4c44a8586995fab2b8cd1e
```

출력로그

```
[root@k8s-workernode3 ~]# kubeadm join 172.16.2.101:6443 --token ty27yj.vvgzmn7ifenfmh6f \
>     --discovery-token-ca-cert-hash sha256:49c44083ec868ba9a5481652b2887cade71f6cfd9a4c44a8586995fab2b8cd1e
W0407 21:04:07.668987   18011 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.17" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

#### 대상서버 : k8s-master1

- Worker nodes join 확인

```
kubectl get nodes -o wide
```

출력로그

```
[root@k8s-master1 k8s_src]# kubectl get nodes -o wide
NAME              STATUS   ROLES    AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
k8s-master1       Ready    master   3h7m    v1.17.4   172.16.2.101   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.4
k8s-workernode1   Ready    <none>   11m     v1.17.4   172.16.2.111   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.4
k8s-workernode2   Ready    <none>   5m55s   v1.17.4   172.16.2.112   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.4
k8s-workernode3   Ready    <none>   96s     v1.17.4   172.16.2.113   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.4
```

- k8s cluster 및 resources 정보 확인

```
kubectl cluster-info
kubectl get componentstatuses
kubectl get po --all-namespaces -o wide
kubectl get deployments --all-namespaces -o wide
kubectl get daemonset --all-namespaces -o wide
kubectl get service --all-namespaces -o wide
```

출력로그

```
root@k8s-master1 k8s_src]# kubectl cluster-info
Kubernetes master is running at https://172.16.2.101:6443
KubeDNS is running at https://172.16.2.101:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```
[root@k8s-master1 k8s_src]# kubectl get componentstatuses
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}
```

```
[root@k8s-master1 k8s_src]# kubectl get po --all-namespaces -o wide
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE     IP              NODE              NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-5554fcdcf9-bccf8   1/1     Running   0          48m     10.10.159.130   k8s-master1       <none>           <none>
kube-system   calico-node-knxwq                          1/1     Running   0          48m     172.16.2.101    k8s-master1       <none>           <none>
kube-system   calico-node-mzrg8                          1/1     Running   0          29m     172.16.2.111    k8s-workernode1   <none>           <none>
kube-system   calico-node-p68cz                          1/1     Running   0          23m     172.16.2.112    k8s-workernode2   <none>           <none>
kube-system   calico-node-r778n                          1/1     Running   0          19m     172.16.2.113    k8s-workernode3   <none>           <none>
kube-system   coredns-6955765f44-fcvwn                   1/1     Running   0          3h25m   10.10.159.129   k8s-master1       <none>           <none>
kube-system   coredns-6955765f44-pjn89                   1/1     Running   0          3h25m   10.10.159.131   k8s-master1       <none>           <none>
kube-system   etcd-k8s-master1                           1/1     Running   0          3h25m   172.16.2.101    k8s-master1       <none>           <none>
kube-system   kube-apiserver-k8s-master1                 1/1     Running   0          3h25m   172.16.2.101    k8s-master1       <none>           <none>
kube-system   kube-controller-manager-k8s-master1        1/1     Running   0          3h25m   172.16.2.101    k8s-master1       <none>           <none>
kube-system   kube-proxy-4mz2x                           1/1     Running   0          23m     172.16.2.112    k8s-workernode2   <none>           <none>
kube-system   kube-proxy-8m7dh                           1/1     Running   0          3h25m   172.16.2.101    k8s-master1       <none>           <none>
kube-system   kube-proxy-r6nvg                           1/1     Running   0          29m     172.16.2.111    k8s-workernode1   <none>           <none>
kube-system   kube-proxy-txkvf                           1/1     Running   0          19m     172.16.2.113    k8s-workernode3   <none>           <none>
kube-system   kube-scheduler-k8s-master1                 1/1     Running   0          3h25m   172.16.2.101    k8s-master1       <none>           <none>
```

```
[root@k8s-master1 k8s_src]# kubectl get deployments --all-namespaces -o wide
NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                IMAGES                            SELECTOR
kube-system   calico-kube-controllers   1/1     1            1           49m     calico-kube-controllers   calico/kube-controllers:v3.13.2   k8s-app=calico-kube-controllers
kube-system   coredns                   2/2     2            2           3h26m   coredns                   k8s.gcr.io/coredns:1.6.5          k8s-app=kube-dns
```

```
[root@k8s-master1 k8s_src]# kubectl get daemonset --all-namespaces -o wide
NAMESPACE     NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE     CONTAINERS    IMAGES                          SELECTOR
kube-system   calico-node   4         4         4       4            4           kubernetes.io/os=linux        50m     calico-node   calico/node:v3.13.2             k8s-app=calico-node
kube-system   kube-proxy    4         4         4       4            4           beta.kubernetes.io/os=linux   3h26m   kube-proxy    k8s.gcr.io/kube-proxy:v1.17.4   k8s-app=kube-proxy
```

```
[root@k8s-master1 k8s_src]# kubectl get service --all-namespaces -o wide
NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE     SELECTOR
default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  3h27m   <none>
kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3h27m   k8s-app=kube-dns
```

### Dashboard 설치
- https://github.com/kubernetes/dashboard
- releases 클릭
- Dashboard version과 k8s version 호환성 확인

~~kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc7/aio/deploy/recommended.yaml~~

```
curl https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc7/aio/deploy/recommended.yaml -o $HOME/k8s_src/dashboard_v2.0.0-rc7.yaml
cd $HOME/k8s_src
kubectl apply -f dashboard_v2.0.0-rc7.yaml
```

출력 로그

```
[root@k8s-master1 k8s_src]# kubectl apply -f dashboard_v2.0.0-rc7.yaml 
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

- Dashboard 서비스 확인
  - NAMESPACE에 kubernetes-dashboard 추가 확인
  - Pods 생성 확인
  - Deployment 확인
  - Service 확인

```
kubectl get pods --namespace kubernetes-dashboard
kubectl get deploy -n kubernetes-dashboard
kubectl get service -n kubernetes-dashboard
```

출력 로그
```
[root@k8s-master1 k8s_src]# kubectl get po --all-namespaces
NAMESPACE              NAME                                        READY   STATUS    RESTARTS   AGE
kube-system            calico-kube-controllers-5554fcdcf9-bccf8    1/1     Running   0          3h38m
kube-system            calico-node-knxwq                           1/1     Running   0          3h38m
kube-system            calico-node-mzrg8                           1/1     Running   0          3h19m
kube-system            calico-node-p68cz                           1/1     Running   0          3h13m
kube-system            calico-node-r778n                           1/1     Running   0          3h9m
kube-system            coredns-6955765f44-fcvwn                    1/1     Running   0          6h15m
kube-system            coredns-6955765f44-pjn89                    1/1     Running   0          6h15m
kube-system            etcd-k8s-master1                            1/1     Running   0          6h15m
kube-system            kube-apiserver-k8s-master1                  1/1     Running   0          6h15m
kube-system            kube-controller-manager-k8s-master1         1/1     Running   0          6h15m
kube-system            kube-proxy-4mz2x                            1/1     Running   0          3h13m
kube-system            kube-proxy-8m7dh                            1/1     Running   0          6h15m
kube-system            kube-proxy-r6nvg                            1/1     Running   0          3h19m
kube-system            kube-proxy-txkvf                            1/1     Running   0          3h9m
kube-system            kube-scheduler-k8s-master1                  1/1     Running   0          6h15m
kubernetes-dashboard   dashboard-metrics-scraper-b68468655-tr9d9   1/1     Running   0          111s
kubernetes-dashboard   kubernetes-dashboard-64999dbccd-s2fmv       1/1     Running   0          111s

[root@k8s-master1 k8s_src]# kubectl get pods -n kubernetes-dashboard
NAME                                        READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-b68468655-tr9d9   1/1     Running   0          2m10s
kubernetes-dashboard-64999dbccd-s2fmv       1/1     Running   0          2m10s

[root@k8s-master1 k8s_src]# kubectl get deploy --all-namespaces
NAMESPACE              NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
kube-system            calico-kube-controllers     1/1     1            1           3h39m
kube-system            coredns                     2/2     2            2           6h16m
kubernetes-dashboard   dashboard-metrics-scraper   1/1     1            1           2m38s
kubernetes-dashboard   kubernetes-dashboard        1/1     1            1           2m38s

[root@k8s-master1 k8s_src]# kubectl get deploy -n kubernetes-dashboard -o wide
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                  IMAGES                                SELECTOR
dashboard-metrics-scraper   1/1     1            1           8m31s   dashboard-metrics-scraper   kubernetesui/metrics-scraper:v1.0.4   k8s-app=dashboard-metrics-scraper
kubernetes-dashboard        1/1     1            1           8m31s   kubernetes-dashboard        kubernetesui/dashboard:v2.0.0-rc7     k8s-app=kubernetes-dashboard

[root@k8s-master1 k8s_src]# kubectl get service -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
dashboard-metrics-scraper   ClusterIP   10.108.147.129   <none>        8000/TCP   11m
kubernetes-dashboard        ClusterIP   10.103.226.218   <none>        443/TCP    11m
```

- Dashboard service 계정 생성
  - dashboard-adminuser.yaml 생성

```bash
vi dashboard-adminuser.yaml
===========================
# Create Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---

# Create ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
> NOTE: apiVersion of ClusterRoleBinding resource may differ between Kubernetes versions. Prior to Kubernetes v1.8 the apiVersion was rbac.authorization.k8s.io/v1beta1<br>
> TIP: apiVersion 확인방법<br>
> kubectl explain ClusterRoleBinding

```
kubectl apply -f dashboard-adminuser.yaml
```

출력 로그

```
[root@k8s-master1 k8s_src]# kubectl apply -f dashboard-adminuser.yaml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

- admin-user 계정의 접속 token 값 확인

```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

출력 로그

```
[root@k8s-master1 k8s_src]# kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
Name:         admin-user-token-b4cx9
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: f8476f13-5ae0-4594-afb3-d5e7c6759765

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkdYTWFIdjBHMk1Ld3dxV2VqZDFaOUdVQTgtWFRIeVptUVVUME95MW1oeEUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWI0Y3g5Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmODQ3NmYxMy01YWUwLTQ1OTQtYWZiMy1kNWU3YzY3NTk3NjUiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.mgh2AXGMOR40oMNs_0opb-aUSHHkCa3PQD8p75p3quj9RVmBh_MC4eHxpizVQkCdc95MHGRXHfTpfMNbOH-9ZeFchRnum34ywCZ6JoifWOUJtPRN0_vLm5tZbP1JnvXLeuIF3JhhQTFWe9ZzAJg3wH8_4fRuRVmV-JnWFOG1WhpcFkuN64kxNZbpd5KHva0zcm5EbuP8k1zxZMTrMh-y7esUpvTxcGhSwU_1MJmKbiXT73uYpWncQX4LIKITEV20YrSdTmJetUCNYtG_doEtuHyuorUTwjGO1fH8KLJGgYYhxYjY5JLEZ440ceXe1zQxRtlpYt2bNX6FFP-PurIxhA
```

- Tunneling 설정 및 작업 순서
  - 기본적으로 Dashboard는 localhost에서만 접속 가능(외부에서 접속 불가)
  - putty의 tunnel을 이용하여 원격지pc 에서도 접속 가능 하도록 한다
  - putty 터널링 설정 -> 서버 접속 -> kubectl proxy 실행 -> 원격지 pc에서 웹브라우저 접속 -> 베어러(Bearer) 토큰 입력

![](images/putty-tunnel-1.png "")

```
kubectl proxy --port=8001
```

- 브라우저에서 접속

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

### 리소스 모니터링 도구 설치
- https://kubernetes.io/ko/docs/tasks/debug-application-cluster/resource-usage-monitoring/
- https://github.com/kubernetes-sigs/metrics-server
- heapster는 metrics-server에 통합되었음
- kubectl top pod, kubectl top node, Dashboard상에서 Graph출력
- Horizontal Pod Autoscaler(HPA)가 메트릭을 수집할 때 metrics-server api를 사용하여 Replicaset, deployment, statefull의 pod를 autoscale 한다

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
```

```
cd $HOME/k8s_src
git clone https://github.com/kubernetes-sigs/metrics-server.git
cd metrics-server/deploy/kubernetes
kubectl apply -f ./
```

- Resource 모니터링(초기 수집시 약간이 시간 필요)

```
kubectl top node
kubectl top pod -A
k8s Dashboard 접속 후 Cluster > node 선택후 CPU, Mem usage 확인
```

- Resource 모니터링시 반응이 없으면 metrics-server POD log 확인 후 아래의 작업 실행
- args에 "--kubelet-insecure-tls", "-- kubelet-preferred-address-types" 추가

```
kubectl -n kube-system logs $(kubectl get pods --namespace=kube-system -l k8s-app=metrics-server -o name)

vi metrics-server-deployment.yaml
=================================
args:
  - --cert-dir=/tmp
  - --secure-port=4443
  - --kubelet-insecure-tls
  - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

- metrics-server re-deploy

```
kubectl apply -f ./
```

- Resource 모니터링

```
kubectl top node
kubectl top pod -A
k8s Dashboard 접속 후 Cluster > node 선택시 CPU, Mem usage 확인
```

## Pesistent Volume(Storage) 구성
### NFS 구성
#### 대상서버 : k8s-nfs1

- nfs-server 설치

```
yum install nfs-utils
rpcinfo -p
systemctl start nfs && systemctl enable nfs
rpcinfo -p
```

- nfs-server 설정 및 디렉토리 생성(옵션확인필요)
- 방화벽 설정 필요시 /etc/sysconfig/nfs 포트 고정

```
vi /etc/exports
===============
/nfs-data	10.0.0.0/16(rw,sync,no_subtree_check,no_root_squash)
```

```
mkdir /nfs-data
showmount -e 10.0.0.201
exportfs -rav
exportfs -v
```

---
컨테이너 runtime 설치
> https://v1-17.docs.kubernetes.io/ko/docs/setup/production-environment/container-runtimes/  
> https://docs.docker.com/install/

쿠버네티스 설치 tool
> https://v1-17.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/


컨테이너 네트워크 인터페이스(CNI)
> https://zetawiki.com/wiki/%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88_%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4_CNI <br>
> https://medium.com/@lhs6395/container-network-interface-e36b199be83f<br>
> https://kubernetes.io/docs/concepts/cluster-administration/networking/  
> https://gowithoss.blogspot.com/2019/10/blog-post.html  
> https://docs.google.com/spreadsheets/d/1qCOlor16Wp5mHd6MQxB5gUEQILnijyDLIExEpqmee2k/edit#gid=0

쿠버네티스 설치
> https://www.cubrid.com/blog/3820603

쿠버네티스 명령어 정리
- https://github.com/freepsw/kubernetes_exercise
- https://gist.github.com/edsiper/fac9a816898e16fc0036f5508320e8b4

쿠버네티스 리소스 모니터링
- https://gruuuuu.github.io/cloud/monitoring-k8s1/
