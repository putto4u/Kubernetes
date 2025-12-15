 **하이브리드 클라우드 전략 및 클러스터 구축**(python, pyenv, yaml, docker, k8s, network, 보안)까지 고려한 **모든 필수 구성 요소, 상세 설치 명령, 보안 강화, 네트워크 전략, 그리고 복제 전 초기화 작업**을 하나의 완벽한 최종 가이드라인으로 통합하여 정리해 드립니다.

---

##🚀 최종 골든 이미지(Golden Image) 구축 가이드: 클라우드 & 클러스터 통합

###I. 📋 골든 이미지 구축 핵심 요약 (상세 설명)

| 단계 | 항목 | 목적 및 **주요 작업 내용** | 복제 전 수행 여부 |
| --- | --- | --- | --- |
| **1단계** | 기본 시스템 및 사용자 설정 | OS 안정화 및 원격 접속 환경 구축. SSH 서버, `git` 설치 및 기본 설정 포함. | **✅ 필수** |
| **2단계** | 운영체제 기본 보안 강화 | **클러스터 통신 포트 사전 허용**, UFW 활성화, 패스워드 정책 강화 등 보안 설정. | **✅ 필수** |
| **3단계** | YAML/JSON 도구 및 pyenv 설치 준비 | **파이썬 빌드 의존성** 확보 및 `pyenv` 쉘 환경 설정. `yq`, `jq` 설치 포함. | **✅ 필수** |
| **4단계** | pyenv 환경 및 라이브러리 구축 | 다중 파이썬 버전 관리 환경 구축. **파이썬 버전 및 핵심 라이브러리** 사전 설치. | **✅ 필수** |
| **5단계** | 도커 엔진 및 네트워크 전략 | 컨테이너 환경 구축 및 **클라우드 IP 충돌 방지 전략** 수립. | **✅ 필수** |
| **6단계** | **시스템 초기화 및 정리** | 복제본 간의 충돌 방지 및 용량 최적화. 로그, 히스토리, **네트워크 고유 설정** 제거. | **🛑 필수** |

---

###II. 🚀 상세 환경 구성 및 설치 명령 가이드####
1단계: 기본 시스템 및 사용자/접근 설정 (필수)

| 항목 | 목적 | 명령어 (Ubuntu 22.04 LTS 기준) |
| --- | --- | --- |
| **시스템 업데이트** | 최신 패키지 리스트 및 시스템 업데이트 | `$ sudo apt update && sudo apt upgrade -y` |
| **기본 도구 설치** | `vim`, `curl`, `wget`, `build-essential` 등 필수 도구 | `$ sudo apt install -y vim curl wget build-essential` |
| **SSH 서버 설치/가동** | 원격 접속 환경 마련 | `$ sudo apt install -y openssh-serve`r <br> `$ sudo systemctl enable ssh && sudo systemctl start ssh` |
| **`git` 설치 및 기본 설정** | 버전 관리 도구 및 사용자 정보 설정 | `$ sudo apt install -y git` <br> `$ git config --global user.name "Your Name"   $ git config --global user.email "your.email@example.com"` |
| **Guest Additions** | 호스트-게스트 간 기능 활성화 | *버추얼박스 메뉴에서 `장치` > `Guest Additions CD 이미지 삽입` 후 VM 내부에서 설치* |

####2단계: 운영체제 기본 보안 강화 (클러스터 준비)골든 이미지에 UFW를 설정할 때는 향후 쿠버네티스 클러스터 구축에 필요한 포트를 미리 개방하여 확장성을 확보합니다.

| 항목 | 목적 | 명령어 (Ubuntu UFW 기준) |
| --- | --- | --- |
| **방화벽 기본 정책** | 나가는 트래픽 허용, 들어오는 트래픽 기본 차단 | `bash$ sudo ufw default allow outgoing`   <br> `$ sudo ufw default deny incoming` |
| **SSH 포트 허용** | 원격 관리(22번) 포트 허용 | `$ sudo ufw allow 22/tcp` |
| **쿠버네티스 포트 사전 허용** | 향후 클러스터 통신을 위한 필수 포트 개방 | `$  # Control Plane (Master/Manager) 포트   $ sudo ufw allow 6443/tcp # K8s API 서버   $ sudo ufw allow 2379:2380/tcp # etcd 서버   $ sudo ufw allow 10259/tcp # Scheduler # Worker Node 포트  $ sudo ufw allow 10250/tcp # Kubelet API   $ sudo ufw allow 30000:32767/tcp # NodePort 범위` |
| **방화벽 활성화** | 규칙 적용 | `$ sudo ufw enable   $ sudo ufw status` |
| **패스워드 정책 강화** | 사용자 패스워드 복잡도 및 주기 설정 | * `/etc/login.defs` 및 `/etc/pam.d/common-password` 파일 수정하여 설정 (매뉴얼 수정 권장)* |

####3단계: YAML/JSON 도구 및 pyenv 설치 준비1. **YAML/JSON 도구 설치:**
| 도구 | 목적 | 명령어 |
| :--- | :--- | :--- |
| **yq & jq** | 설정 파일 파싱 및 수정 유틸리티 | `$ sudo wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64   $ sudo chmod a+x /usr/local/bin/yq   $ sudo apt install -y jq   ` |
2. **pyenv 빌드 의존성 설치:**
```$ sudo apt install -y libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev   

```


3. **pyenv 설치:**
```bash
# pyenv 설치
$ curl https://pyenv.run | $
# 쉘 설정 파일 (.bashrc)에 pyenv 환경 변수 등록 및 적용
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
$ echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(pyenv init -)"' >> ~/.bashrc
$ source ~/.bashrc 

```



####4단계: pyenv 환경 및 라이브러리 구축1. **가상 환경 도구 설치:**
```bash
# pyenv-virtualenv 설치 및 설정
$ git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
$ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
$ source ~/.bashrc 

```


2. **파이썬 버전 및 핵심 라이브러리 설치:**
```bash
# 파이썬 버전 설치 및 전역 설정
$ pyenv install 3.10.12 # 예시 버전
$ pyenv global 3.10.12 

# 필수 라이브러리 설치 (전역 버전 환경에 설치)
$ pip install numpy pandas scikit-learn

```



####5단계: 도커 엔진 및 네트워크 전략 (하이브리드 클라우드 고려)1. **도커 엔진 설치:**
```bash
# 도커 설치 (GPG 키 및 리포지토리 추가 생략)
$ sudo apt update
$ sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 현재 사용자에게 도커 그룹 권한 부여
$ sudo usermod -aG docker $USER

```


2. **도커 브릿지 대역 점검/변경 (⭐ 하이브리드 클라우드 핵심):**
**실전 팁:** 도커 기본 대역(`172.17.0.0/16`)이 향후 AWS VPC 등 클라우드 사설망과 **겹칠 경우** 라우팅 충돌이 발생합니다. 복제 전에 충돌 방지용 새로운 대역(예: 172.50.0.0/16)으로 변경하는 것을 강력히 권장합니다.
```bash
# [예시] 도커 대역 변경
$ sudo nano /etc/docker/daemon.json
# 내용:
{
  "bip": "172.50.0.1/16"
}
$ sudo systemctl restart docker

```



---

###III. 🛑 복제 전에 반드시 해야 할 시스템 초기화 (Critical)| 작업 항목 | 목적 | 명령어 |
| --- | --- | --- |
| **VM 네트워크 설정** | **DHCP 유지** | VM 네트워크는 **DHCP**로 설정하여 복제 후 고유 IP를 할당받도록 합니다. **고정 IP 설정은 복제 후** VM 역할(마스터/워커)에 따라 진행합니다. |
| **네트워크 설정 초기화** | 고유 설정 제거 | *OS 설정 파일(예: `/etc/netplan` 또는 `/etc/network/interfaces`)을 검토하여 **MAC 주소나 고정 IP 설정이 남아있지 않도록** DHCP로 되돌려야 합니다.* |
| **로그 및 캐시 정리** | 이미지 용량 최적화 및 불필요한 데이터 제거 | `$ sudo apt clean   $ sudo apt autoremove -y   $ sudo rm -rf /var/log/* /tmp/*   ` |
| **쉘 히스토리 제거** | 이전 작업 기록 삭제 | `$ history -c && exit   ` |

VM을 종료하고 복제하면, **보안 강화, 도구 준비, 그리고 클라우드 환경과의 호환성**까지 모두 갖춘 완벽한 골든 이미지를 확보할 수 있습니다.
