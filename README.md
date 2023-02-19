# CICD

jekins

1번째 실습
흐름 : 도커 젠킨스 -> SSH 도커 톰캣에 젠킨스 패키징 파일 빌드 배포

1. docker image로 젠킨스 설치 (도커기때문에 이미 설치된 상태 jdk 설정 불필요)
2. item 생성
3. 깃허브 플러그인 설치 여부 확인
4. 대쉬보드 - globalconfiguration에 git repo 연결 (도커 컨테이너 내부에서 깃 버전 확인 git --version)
5. 빌드 툴 gradle -v로 확인 없는경우 globalconfiguration에서 설정 필요
6. war를 사용하기에 tomcat docker 생성
7. 톰캣(user.xml, context.xml)수정하고 jekins에 credential 설정
8. jenkins 빌드후 docker tomcat ssh에 이동후 서버에 반영

2번째 실습
흐름 : 도커 젠킨스 -> SSH 도커 리눅스 내부에 도커에 젠킨스 패키징 파일 수신 -> 이미지 생성

1. Jenkins Publish Over SSH 플러그인 설치
2. Jenkins Configure System -> Publish Over SSH 설정 (도커)
3. second 디렉토리내의 ubuntu dockerfile과 docker compose 실행 초기 PW: root로 설정
4. 다시 2번으로 돌아와서 젠킨스에 SSH Servers 설정
   jenkins 도커 권한 (docker exec -u 0 -it mycontainer bash)
5. docker in docker로 (우분투에 도커 설치)
   systemctl start docker 실행시 다음과 같은 에러가 발생하는 경우
   --
   System has not been booted with systemd as init system (PID 1). Can't operate.
   --
   1.sudo apt-get update && sudo apt-get install -yqq daemonize dbus-user-session fontconfig

2.sudo daemonize /usr/bin/unshare --fork --pid --mount-proc /lib/systemd/systemd --system-unit=basic.target

3.exec sudo nsenter -t $(pidof systemd) -a su - $LOGNAME

4.snap version 없는경우 sudo apt-get install snapd 인스톨

5. systemctl status 확인

6. 젠킨스 Send build artifacts over SSH 설정

7. 8081로 접속

주의점)
현재 상태

호스트 PC 도커
젠킨스 PORT 8080:8080
우분투 PORT 8081:8080, 10022:22

우분투안에는 도커가 또 존재하기때문에(Docker in Docker) 젠킨스에서 패키징한 파일을 우분투에 SSH 접속을 통해 전달하게 되면 우분투에서 WAR를 외장 톰캣으로 배포하게됨 이때 톰캣을 8080이고 호스트 PC는 8081에서 포워딩 되기때문에 8081로 접속하면 확인이 가능하다.
