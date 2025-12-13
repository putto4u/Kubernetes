# Kubernetes (K8S)


## 사전 준비
1. Oracle VirtualBox 설치(슈퍼바이저)
2. VM(가상머신) 생성 - 서버네임 : puttoSVR01
3. VM에 Ubuntu 리눅스 설치
4. $ sudo apt update
5. $ sudo apt install net-tools       #네트워크 명령어들
6. $ sudo apt install ca-certificates curl  gnupg  lsb-release
      > ca_certificates : 인증서관련패키지  
      > curl : (웹)파일 다운로드 패키지  
      > gnupg : 디지털 서명 패키지  
      > lsb-release : 리눅스 배포판 식별 패키지
7. $ sudo mkdir -m 0755 -p /etc/apt/keyrings     # GPG(GNU Privacy Guard) 키를 사용합니다. 이 키들을 모아두는 저장소
8. 
9. :
10. Python, Pyenv 설치
11. :
12. :
13. 버춸박스에서 VM2,3 복제 - 서버네임 :ㅇㅇsvr02,ㅇㅇsvr03  맥어드레스정책:모든 네트워크 어댑터의 새Mac주소 생성, 완전한 복제 선택
14. vm2, vm3... 각각 설정 변경  - ```svr02$ sudo hostnamectl set-hostname ㅇㅇsvr02 ;  $ cat /etc/hostname ; $ sudo reboot now
15.                           svr02$ ifconfig  ; 
16. 
17. 
