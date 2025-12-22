# Kubernetes (K8S)


## 사전 준비 - 골든 이미지를 완성하고 vm을 복제한 후 쿠버네티스 설치에 들어가기 위한 사전 단계
1. Oracle VirtualBox 설치(슈퍼바이저)
2. VM(가상머신) 생성 - 서버네임 : puttoSVR01
3. VM에 Ubuntu 리눅스 설치
4. apt update
    ```bash
    $ sudo apt clean                       # update 실행시 오류가 발생할 때 사용
    $ sudo rm -rf /var/lib/apt/lists/*     # update 실행시 오류가 발생할 때 사용
    $ sudo apt update
    ```  

5.  ```bash
    $ sudo apt install net-tools       # ifconfig 등 네트워크 명령어 패키지 설치
    ```
6.  ```bash
    $ sudo apt install  ca-certificates curl  gnupg  lsb-release  build_essential
    ```  
        > ca_certificates : 인증서관련패키지  
        > curl : (웹)파일 다운로드 패키지  
        > gnupg : 디지털 서명 패키지  
        > lsb-release : 리눅스 배포판 식별 패키지
        > build-essential : 소프트웨어를 소스코드로부터 직접 컴파일하고 빌드하는 라이브러리 메타 패키지

6. ssh 가동 및 작동 확인
   ```bash
   $ sudo systemctl enable ssh && sudo systemctl start ssh  # 서버 부팅시 ssh 자동 시작, 서버 지금 시작
   $ systemctl list-units --type=service --state=running
   ```
   
7. GPG 키 추가
   ```bash
   $ sudo mkdir -m 0755 -p /etc/apt/keyrings     # GPG(GNU Privacy Guard) 키들을 모아두는 저장소
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg  # GPG키 다운로드 및 추가
   ```
   
8. 도커 리포지토리 설정
    ```bash
    $ echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

9. 도커설치
    ```bash
    $ sudo apt update    ## 필수 실행. 실제업데이트(upgrade)가 아님! 업데이트 버전, 다운로드 url, 목록 정보 등을 업데이트 하는 것임. 
    $ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    Docker 자동완성 설정

    최신 버전의 Docker는 쉘 종류에 맞는 자동완성 스크립트를 생성하는 명령어를 내장하고 있습니다.
   
    1. `bash-completion` 패키지가 필요합니다.
       ```bash
       sudo apt install bash-completion -y
       apt list --installed | grep bash-completion
       ```
       
     2. 자동완성 스크립트를 등록합니다.
        ```bash
        # 스크립트 생성 및 적용
        mkdir -p ~/.local/share/bash-completion/completions
        docker completion bash > ~/.local/share/bash-completion/completions/docker

        # 바로 적용
        source ~/.bashrc
        ```

## 10. pyenv 설치(Python 설치 전 작업)
    ## 사전 설치 패키지들

    $ sudo apt update
    $ sudo apt install -y build-essential libssl-dev zlib1g-dev \
    libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev


    ## pyenv 설치

    $ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
    # 또는 
    $ curl https://pyenv.run | bash   # git을 사용할 수도 있지만 git을 사용하고 싶지 않을 경우 사용한다.


    ## ~/.bashrc 파일의 맨 끝에 추가    

    $ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
    $ echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
    $ echo 'eval "$(pyenv init -)"' >> ~/.bashrc

    ## 또는

    $ vim .bashrc    # .bashrc 파일을 열어 맨 마지막에 아래 세 줄을 추가한다.
      #명령 모드 'G' -> 'o':   G - 마지막줄 이동  o - 다음 줄에 입력시작
      #또는, 명령모드에서 ':set mouse=a'(ex모드) 를 입력하면 마우스로 커서를 이동할 수 있음.
      
      export PYENV_ROOT="$HOME/.pyenv"
      export PATH="$PYENV_ROOT/bin:$PATH"
      eval "$(pyenv init -)"
     :wq!   # 저장 후 종료


    ## 변경 사항 적용
    $ source ~/.bashrc  # 쉘 재시작,  .bashrc를 새로 적용
    # 또는
    $ exec $SHELL

    ## 설치 확인
    $ pyenv   # 설치 확인
    :  # 버전 및 명령어 리스트 출력


    
## 11. Python 설치 ( 반드시 pyenv로 설치!!)
   ```bash
   $ /usr/bin/python3 --version   # 시스템파이썬 버전 확인 -> 3.12.3
   $ apt show python3    # apt install python3 이라 명령시 설치될 최신 배포 버전을 알려줌. 하지만 반드시 pyenv로 설치하도록 합니다.

   $ sudo apt update
   $ sudo apt install -y build-essential libssl-dev zlib1g-dev \    ## 앞서 설치 했다면 생략, 제대로 설치가 안되어 있으면 파이썬 설치시 오류 발생. 
     libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
     libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
   $ pyenv install 3.12.12  # pyenv로 apt가 설치할 수 없는 버전도 설치할 수 있으며, 여러 환경에서 설치할 때마다 다른 버전을 설치하고 사용시마다 변경,관리할 수 있습니다.
   ## 성공시 4줄 정도의 짧은 버전 정보가 나옴, 오류 발생시 긴 문장 출력

   ## 오류 발생 시
   # pyenv cache 정리 (선택 사항이지만 권장)
   $ rm -rf ~/.pyenv/cache

   # pyenv 빌드 환경 변수 초기화 (필요하다면)
   $ unset LDFLAGS CFLAGS

   # Python 3.12.12 설치 재시도
   $ pyenv install 3.12.12
   ```
   ```bash
   $ pyenv versions  #또는 python --version   ## 설치 확인, 버전출력
   ## 안나오면
   $ pyenv global 3.12.12     ## 사용자의 모든 환경에서 3.12.12를 사용하도록 설정됨
   $ pyenv versions  #또는 python --version  ## 버전 출력
   ```
12. dfㄴㅇㄹ
13. fgㅇㄴㄹ
14. ㄴㅇㄹ
15. 
16. :
17. :
18. 버춸박스에서 VM2,3 복제 - 서버네임 :ㅇㅇsvr02,ㅇㅇsvr03  맥어드레스정책:모든 네트워크 어댑터의 새Mac주소 생성, 완전한 복제 선택
19. vm2, vm3... 각각 설정 변경  - ```svr02$ sudo hostnamectl set-hostname ㅇㅇsvr02 ;  $ cat /etc/hostname ; $ sudo reboot now
20.                           svr02$ ifconfig  ; 
21. 
22. 
