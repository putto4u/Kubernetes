## 🚨 실전 팁: VM 복제 및 설정 순서 (IT 전문가의 골든 이미지 전략)

**다중 노드 환경 구축 시 효율성과 안정성** 측면에서 vm복제 단계 결정은 매우 중요한 포인트입니다. 결론부터 말씀드리면, 쿠버네티스 설치를 거의 다 한 후에 호스트네임이나 네트워크 설정을 바꾸는 것은 **매우 위험하며 심각한 오류**를 유발합니다.

**가장 효율적이고 안정적인 순서**와 그 이유를 명확히 설명해 드립니다.

-----

### 1\. ⚠️ 자주 오해하거나 실수하는 부분: 설정 변경 타이밍

| 구분 | 시나리오 | 발생 문제 |
| :--- | :--- | :--- |
| **권장 순서 (O)** | OS 설치 및 필수 패키지 설치 → **VM 복제** → **호스트네임/IP 변경** → `kubeadm init` 실행 | 안정적인 클러스터 구축 가능 |
| **위험 순서 (X)** | `kubeadm init` 실행 (Master 설정) → **VM 복제** → 호스트네임/IP 변경 | **쿠버네티스 인증서 및 구성 파일이 무효화됨** |

#### 왜 K8s 설치 전에 바꿔야 하는가?

**쿠버네티스 컴포넌트 (특히 `kubeadm` 도구)는 마스터 노드를 설정할 때 호스트네임과 IP 주소를 기반으로 핵심 인증서(Certificates)와 설정 파일을 생성합니다.**

1.  **인증서 문제:** 생성된 인증서의 CN(Common Name) 또는 SAN(Subject Alternative Name)에 **기존의 호스트네임과 IP**가 포함됩니다.
2.  **노드 통신 실패:** 설치 후 호스트네임을 바꾸면, 쿠버네티스 컴포넌트들이 변경된 새 이름을 인식하지 못하고, 통신 시 인증서 검증에 실패하여 **클러스터 작동이 멈춥니다.**
3.  **복구의 어려움:** 결국 `kubeadm reset`을 하고 모든 인증서를 **재생성**해야 하므로, 복사해서 얻으려 했던 시간 절약 효과가 사라지고 훨씬 더 많은 시간을 낭비하게 됩니다.

-----

### 2\. ✅ 전문가의 정석 워크플로우: 골든 이미지 복제

3개의 VM 노드를 만들 때 가장 효율적인 방법은 \*\*"골든 이미지(Golden Image)"\*\*를 만든 후 복제하는 것입니다.

#### 🥇 단계 1: 골든 이미지 준비 (최대 효율)

가장 기본적인 VM(Node 1)에 다음 필수 요소들을 설치하고 설정합니다.

1.  **기본 OS 설치 완료.**
2.  **SSH 서버 설정.** (노드 간 통신 및 외부 접근 용이)
3.  **Docker 및 Containerd 설치 완료.** (단, 도커 저장소 등록까지만 하고, `docker-ce` 설치는 미루거나 설치 후 최종 복제 전에 `apt autoremove`로 정리할 수도 있습니다. 하지만 여기서는 **도커 설치까지 완료**하는 것이 가장 일반적인 실전 팁입니다.)
4.  **쿠버네티스 패키지 저장소 등록 완료.** (`apt update`까지 실행)
5.  **Swap 비활성화 및 Kernel 모듈 로드 설정 완료.** (k8s 필수 조건)

#### 🥈 단계 2: VM 복제 및 고유 ID 변경 (Critical Step)

골든 이미지를 만든 후, 2개 또는 3개로 복제합니다.

| 작업 (복제 직후) | 명령어 및 설명 |
| :--- | :--- |
| **1. 호스트네임 변경** | `sudo hostnamectl set-hostname k8s-master` 또는 `k8s-worker-01` |
| **2. 정적 IP 설정** | 네트워크 설정 파일(`/etc/netplan/` 또는 `/etc/network/interfaces`)을 수정하여 **고유의 Static IP**를 할당합니다. |
| **3. VirtualBox 복제 설정** | 복제 시 **"새로운 MAC 주소 생성"** 옵션을 반드시 선택하여 네트워크 충돌을 방지합니다. |
| **4. `/etc/hosts` 설정** | 모든 노드에서 다른 노드의 이름과 IP를 매핑해 줍니다. <br> (예: `192.168.x.10 k8s-master`) |
| **5. 고유 시스템 ID 제거** | `sudo rm /etc/machine-id` 및 `sudo truncate -s 0 /etc/machine-id` 실행 후 리부팅. (복제된 시스템이 네트워크에서 고유한 식별자를 갖도록 보장) |

#### 🥉 단계 3: 쿠버네티스 설치 (최종)

모든 노드가 고유한 호스트네임과 Static IP를 갖춘 상태에서 쿠버네티스를 설치합니다.

1.  **Master 노드에서:** `sudo kubeadm init --pod-network-cidr=10.244.0.0/16` 등을 실행.
2.  **Worker 노드에서:** Master 노드에서 출력된 `kubeadm join ...` 명령을 실행.

이 순서를 따르면, 복제는 설치 시간 절약에 기여하고, 고유 설정은 클러스터의 안정성과 무결성을 보장합니다.



---


## 🐳 쿠버네티스 (Kubernetes, K8s) 설치 가이드

쿠버네티스 설치는 여러 가지 구성 요소와 사전 작업이 필요하기 때문에 복잡하게 느껴질 수 있습니다. 하지만 과정을 명확하게 나누어 단계별로 진행하면 충분히 쉽게 따라 할 수 있습니다.

IT 전문가로서 **가장 일반적이고 표준적인 설치 방법**인 \*\*`kubeadm`\*\*을 사용하여 **멀티 노드 클러스터**를 구축하는 과정을 자세하게 안내해 드립니다.

우리는 최소 2대의 Linux 서버(Master 1대, Worker N대)가 준비되었다고 가정하고 설명합니다.

-----

## I. 🛠️ 사전 작업: 모든 노드에 적용 (Pre-requisites)

이 단계는 **Master 노드와 모든 Worker 노드**에서 동일하게 진행해야 합니다.

### 1\. 호스트 이름 설정 및 통신 확인 (Hostname & Network)

각 노드를 식별하기 쉽도록 호스트 이름을 설정합니다.

```bash
# 1. 호스트 이름 설정 (각 노드에 맞게 변경)
sudo hostnamectl set-hostname "master-node" # Master 노드에서 실행
# 또는
sudo hostnamectl set-hostname "worker-node-1" # Worker 노드에서 실행

# 2. /etc/hosts 파일에 모든 노드의 IP와 호스트 이름 등록 (필수)
# 예시:
# 192.168.1.10 master-node
# 192.168.1.11 worker-node-1
sudo vi /etc/hosts
```

### 2\. 방화벽 비활성화 또는 규칙 설정 (Firewall)

클러스터 내부 통신을 원활하게 하기 위해 방화벽을 비활성화하거나, 필요한 포트를 개방해야 합니다. **실습 환경에서는 비활성화를 권장**합니다.

```bash
# 방화벽 비활성화 (UFW 기준)
sudo systemctl disable ufw
sudo systemctl stop ufw
```

### 3\. 스왑 비활성화 (Disable Swap)

쿠버네티스는 안정적인 성능을 위해 메모리 스와핑(Swapping)을 허용하지 않습니다.

```bash
# 1. 스왑 비활성화
sudo swapoff -a

# 2. 재부팅 후에도 스왑이 비활성화되도록 /etc/fstab 파일을 수정
# /etc/fstab 파일에서 'swap'이 포함된 라인을 찾아 주석 처리(#)합니다.
sudo vi /etc/fstab 
```

### 4\. 컨테이너 런타임 설치 (Container Runtime)

Docker와 같은 \*\*컨테이너 런타임(Container Runtime)\*\*이 필요합니다. 쿠버네티스 최신 버전에서는 **Containerd**를 표준으로 사용합니다.

```bash
# 1. Containerd 설치 및 설정
# 2단계 Docker 설치에서 이미 containerd.io가 설치되었을 수 있습니다.
# 만약 설치되어 있지 않다면, Docker 설치 섹션을 참고하여 설치를 완료합니다.

# 2. Containerd 설정 파일 생성
sudo containerd config default | sudo tee /etc/containerd/config.toml >/dev/null

# 3. systemd cgroup 드라이버 설정 변경 (Containerd의 기본 설정은 systemd가 아닐 수 있음)
# /etc/containerd/config.toml 파일 내에서 [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options] 섹션을 찾아
# SystemdCgroup = true 로 변경
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# 4. Containerd 재시작
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 5\. 커널 모듈 로드 및 네트워크 설정 (Kernel & Networking)

쿠버네티스가 필요한 커널 모듈을 로드하고, `iptables`가 브릿지된 트래픽을 처리하도록 설정합니다.

```bash
# 1. 필요한 커널 모듈 로드
sudo modprobe overlay
sudo modprobe br_netfilter

# 2. 커널 파라미터 설정 (재부팅 후에도 유지되도록 설정)
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

# 3. 설정 적용
sudo sysctl --system
```

### 6\. K8s 핵심 패키지 설치: kubeadm, kubelet, kubectl

쿠버네티스 클러스터 설치 및 관리에 필요한 세 가지 핵심 도구를 설치합니다.

1.  **GPG 키 및 저장소 추가:**

    ```bash
    # 쿠버네티스 GPG 키 등록
    sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    # 쿠버네티스 APT 저장소 추가
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

2.  **설치 및 버전 고정:**

    ```bash
    sudo apt update
    # kubelet, kubeadm, kubectl 설치 (원하는 버전으로 변경 가능, v1.28 예시)
    sudo apt install -y kubelet=1.28.0-1.1 kubeadm=1.28.0-1.1 kubectl=1.28.0-1.1

    # 패키지 버전이 업데이트되지 않도록 고정 (필수)
    sudo apt-mark hold kubelet kubeadm kubectl
    ```

-----

## II. 🚀 설치 작업: Master 노드에서 진행

모든 사전 작업이 완료되었다면, **오직 Master 노드에서만** 클러스터를 초기화합니다.

### 1\. 클러스터 초기화 (Master Initialization)

`kubeadm init` 명령으로 쿠버네티스 컨트롤 플레인(Control Plane)을 시작합니다.

```bash
# [POD_CIDR]은 설치할 CNI (Calico 등)에 따라 다름. Calico의 기본값인 192.168.0.0/16 예시.
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

  * **실행 후:** 명령이 완료되면, 성공 메시지와 함께 **두 가지 중요한 정보**가 출력됩니다.
    1.  **kubectl 설정 명령:** 일반 사용자 계정으로 클러스터를 관리하기 위한 설정 명령어
    2.  **Worker Join Token:** Worker 노드를 클러스터에 연결할 때 필요한 토큰 명령어

### 2\. 일반 사용자로 클러스터 설정

출력된 명령어를 실행하여 현재 사용자 계정이 `kubectl`을 사용하여 클러스터를 관리할 수 있도록 설정합니다.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 3\. CNI (Container Network Interface) 설치

노드 간 통신과 파드(Pod) 간 네트워킹을 위해 네트워크 플러그인(CNI)을 설치해야 합니다. **Calico**를 가장 많이 사용합니다.

```bash
# Calico 설치 (Manifest 파일을 사용하여 적용)
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

### 4\. 클러스터 상태 확인

모든 시스템 파드(Pod)가 정상적으로 실행되고 있는지 확인합니다.

```bash
# 모든 시스템 Pod 확인 (Running 상태가 될 때까지 잠시 기다립니다.)
kubectl get pods --all-namespaces

# Master 노드의 상태 확인 (Ready 상태가 되어야 합니다.)
kubectl get nodes
```

-----

## III. 🔗 연동 작업: Worker 노드에서 진행

Master 노드 초기화 시 출력된 **Join Token 명령어**를 사용하여 Worker 노드를 클러스터에 연결합니다.

### 1\. Worker 노드 연결 (Join)

Worker 노드에서 다음 명령어를 **`sudo` 권한**으로 실행합니다.

```bash
# Master 초기화 시 출력된 Join Token 명령어 그대로 사용
sudo kubeadm join 192.168.1.10:6443 --token [토큰] \
    --discovery-token-ca-cert-hash sha256:[해시값]
```

### 2\. 최종 확인 (Master 노드에서)

Master 노드로 돌아와 Worker 노드가 성공적으로 연결되었는지 확인합니다.

```bash
kubectl get nodes
```

출력 결과에 `master-node`와 `worker-node-1` 모두 나타나고, **STATUS**가 **`Ready`** 상태이면 클러스터 설치가 성공적으로 완료된 것입니다.

이제 완벽한 멀티 노드 쿠버네티스 클러스터가 구축되었습니다. 쿠버네티스 관리에 필요한 **`kubectl` 사용법**에 대해 더 자세히 알아볼까요?
