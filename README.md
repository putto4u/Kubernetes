# Kubernetes (K8S)


## 사전 준비
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
    

## 10. pyenv 설치(Python 설치 전 작업)
    * 사전 설치 패키지들
    ```bash
    $ sudo apt update
    $ sudo apt install -y build-essential libssl-dev zlib1g-dev \
    libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
    ```

    * pyenv 설치
    ```bash
    $ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
    # 또는 
    $ curl https://pyenv.run | bash   # git을 사용할 수도 있지만 git을 사용하고 싶지 않을 경우 사용한다.
    ```

    * ~/.bashrc 파일의 맨 끝에 추가    
    ```bash 
    $ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
    $ echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
    $ echo 'eval "$(pyenv init -)"' >> ~/.bashrc
    ```
    * 또는
    ```bash
    $ vim .bashrc    # .bashrc 파일을 열어 맨 마지막에 아래 세 줄을 추가한다.
      :
      export PYENV_ROOT="$HOME/.pyenv"
      export PATH="$PYENV_ROOT/bin:$PATH"
      eval "$(pyenv init -)"
     :wq!   # 저장 후 종료
    ```

    * 변경 사항 적용
    ```bash
    $ source ~/.bashrc  # 추가된 .bashrc를 새로 적용
    ```
    ```bash
    $ which pyenv   # 설치 확인
    ```
    
### 11. Python 설치
12. df
13. fg
14. ㄴㅇㄹ
15. 
16. :
17. :
18. 버춸박스에서 VM2,3 복제 - 서버네임 :ㅇㅇsvr02,ㅇㅇsvr03  맥어드레스정책:모든 네트워크 어댑터의 새Mac주소 생성, 완전한 복제 선택
19. vm2, vm3... 각각 설정 변경  - ```svr02$ sudo hostnamectl set-hostname ㅇㅇsvr02 ;  $ cat /etc/hostname ; $ sudo reboot now
20.                           svr02$ ifconfig  ; 
21. 
22. 
