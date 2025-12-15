# Kubernetes (K8S)


## 사전 준비
1. Oracle VirtualBox 설치(슈퍼바이저)
2. VM(가상머신) 생성 - 서버네임 : puttoSVR01
3. VM에 Ubuntu 리눅스 설치
4. ``` $ sudo apt update  ```  
5. ``` $ sudo apt install net-tools       #네트워크 명령어 패키지 설치  ```  
6. ``` $ sudo apt install  ca-certificates curl  gnupg  lsb-release  build_essential ```  
      > ca_certificates : 인증서관련패키지  
      > curl : (웹)파일 다운로드 패키지  
      > gnupg : 디지털 서명 패키지  
      > lsb-release : 리눅스 배포판 식별 패키지
      > build-essential : 소프트웨어를 소스코드로부터 직접 컴파일하고 빌드하는 라이브러리 메타 패키지

7. ``` $ sudo systemctl enable ssh && sudo systemctl start ssh  # 서버 부팅시 ssh 자동 시작, 서버 지금 시작  
       $ systemctl list-units --type=service --state=running ```


8. GPG 키 추가
   ```
   $ sudo mkdir -m 0755 -p /etc/apt/keyrings     # GPG(GNU Privacy Guard) 키를 사용합니다. 이 키들을 모아두는 저장소
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg  # GPG키 다운로드 및 추가
   ```
   
10. 도커 리포지토리 설정
   ```
   $ echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```
11. 도커설치
    ```
    $ sudo apt update    ## 필수 실행. 실제업데이트(upgrade)가 아님! 업데이트 버전, 다운로드 url, 목록 정보 등을 업데이트 하는 것임. 
    $ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    ```
12. :
13. ㄴㅇㄹ
14. Python, Pyenv 설치
15. :
16. :
17. 버춸박스에서 VM2,3 복제 - 서버네임 :ㅇㅇsvr02,ㅇㅇsvr03  맥어드레스정책:모든 네트워크 어댑터의 새Mac주소 생성, 완전한 복제 선택
18. vm2, vm3... 각각 설정 변경  - ```svr02$ sudo hostnamectl set-hostname ㅇㅇsvr02 ;  $ cat /etc/hostname ; $ sudo reboot now
19.                           svr02$ ifconfig  ; 
20. 
21. 
