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
   systemctl start docker 실행시 다음과 같은 에러가 발생하는 경우 (도커는 루트의 권한이 필요하기 때문에 발생하며 큰 보안이슈로 지적을 받고 있음)
   --> System has not been booted with systemd as init system (PID 1). Can't operate.
   soultion
   --> sudo apt-get update && sudo apt-get install -yqq daemonize dbus-user-session fontconfig
   sudo daemonize /usr/bin/unshare --fork --pid --mount-proc /lib/systemd/systemd --system-unit=basic.target
   exec sudo nsenter -t $(pidof systemd) -a su - $LOGNAME
   
6. snap version 없는경우 sudo apt-get install snapd 인스톨

7. systemctl status 확인

8. 젠킨스 Send build artifacts over SSH 설정

9. 8081로 접속

주의점)
현재 상태

호스트 PC 도커
젠킨스 PORT 8080:8080
우분투 PORT 8081:8080, 10022:22

우분투안에는 도커가 또 존재하기때문에(Docker in Docker) 젠킨스에서 패키징한 파일을 우분투에 SSH 접속을 통해 전달하게 되면 우분투에서 WAR를 외장 톰캣으로 배포하게됨 이때 톰캣을 8080이고 호스트 PC는 8081에서 포워딩 되기때문에 8081로 접속하면 확인이 가능하다.

리눅스의 경우 빌드가 패키징, 전송, 도커 띄우기에 성공해도
한참 머물다가 타임아웃이 떨어지는데 아래와같은 에러문구가 발생한다.
Build step 'Send build artifacts over SSH' changed build result to UNSTABLE
이는 출력스트림이 문제로 뒤에 아래와 같이 붙여주면 된다.

> /dev/null 2>&1 &

<!-- docker exec -itu 0 docker-server /bin/bash  -->
<!-- -u 0 이 루트 권한으로 접속하는 명령어 -->
<!-- 원격 접속하기 위한 툴 -->
<!-- apt-get install net-tools vim openssh-server -->
<!-- vim /etc/ssh/sshd_config 에서 PermitRootLogin Yes로 설정-->
<!-- service ssh start -->
<!-- systemctl status sshd active인지 확인 -->

<!-- WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED 오류시 -->
<!-- 기존에 해당 포트로 접속한 ssh 동일 서버가 있어서 해당인증으로 접속하려다가 에러가 발생하는것 인증 정보 삭제후 재 로그인 필요 -->

ANSIBLE

1. 기존 리눅스 이미지를 컨테이너 변경하여 띠우고 commit 해서 ansible-server로 생성 push
2. ssh-key-gen 후 자기자신의 서버와 docker-server에 ssh-copy-id 한다. (로그인을 생략하기위함)
3. ANSILBE 모듈은 멱등성을 사용함 (여러번 실행해도 한번 실행한 것과 같은 결과가 나옴) 여러번 카피하여도 한번 카피한것으로 간주함과 같은 개념
4. ansible all -m ping - all은 지정한 그룹을 의미 -m 모듈 ping은 모듈의 종류를 의미 ansible document에서 모듈 정의 확인 가능
   예시) ansible all -m shell -a "free -h"
   각서버에 shell 명령어 전달하여 메모리 정보 확인
5. ansible을 이용해 젠킨스 빌드후 push한다.
6. ansible에 등록된 호스트에게 push한 이미지를 받아서 build하도록 한다.

KUBERNETES 1.컨테이너화 된 애플리케이션 구동 2.서비스 디스커버리와 로드밸런싱 3.스토리지 오케스트레이션 4.자동화된 롤아웃과 롤백 5.자동화된 빈패킹 6.자동화된 복구 7.시크릿과 구성관리

단점

1. 소스코드 배포X, 빌드X
2. 애플리케이션 레벨 서비스X
3. 로깅, 모니터링 솔루션X
4. 포괄적인 머신설정, 유지보수, 관리, 자동 복구시스템을 제공X

흐름
마스터에 (환경등을 설정 하고 개발자가 코드수정) -> 하위 노드들에 마스터가 수정된 정보 전달 (이때 노드 안에서 관리하는 최소 단위를 pod이라고함) pods 간에 로드밸런싱을 함

pods은 애플리케이션을 위해 상호작용하는 컨테이너의 논리적 조합이다. 이러한 노드들을 클러스터로 구성하여 서버를 안전하게 구성할수있다.

도커 데스크탑
systemctl --user start docker-desktop

서비스 확인
kubectl get services

쿠버네티스는 컨테이너엔진이 작동인중인 경우 사용이가능함
쿠버네티스의 작동중인 최소한의 단위가 pod이므로 우리가 실행하고자하는 어플이 컨테이너로 패키징되어 사용할 수 있는 상태가 되어야한다.
아래 의미는 image nginx를 가져와서 sample-nginx라는 pods를 생성하겠다는 의미이다.
kubectl run sample-nginx --image=nginx --port=80
-> pod/sample-nginx created

생성된 pods들을 확인할때는 kubectl get pods

상세 설정을 확인할때는 kubectl describe pod/sample-nginx

컨테이너 포트와 events 항목에 상태, 로그 확인 가능

deployment pods를 여러개로 스케일업, 히스토리, 스케줄링할때 사용
kubectl create deployment sample-nginx --image=nginx
목록 확인
kubectl get deployments

다시한번 pods을 확인해보면
kubectl get pods
랜덤아이디가 부여된 pods 확인가능

pods 삭제 명령어
kubectl delete pods sample-nginx

삭제 명령어
kubectl delete {pods이름}
deployment된 pods를 삭제시 최소한의 pods를 유지하려하기때문에 삭제해도 새로 생성됨

pods replica 생성 (scale up )
kubectl scale deployment sample-nginx --replicas=2

deployment 삭제
kubectl delete deployments.apps/sample-nginx

yml 설정후 kubectl apply -f sample1.yml 실행

pods 노드 확인
kubectl get pods -o wide

쿠버네티스도 도커와 마찬가지로 터널링 exec로 가능
kubectl exec -it {podsId} -- bin/bash

apt-get update && apt-get install -y curl wget 이후
내부 hostname -i로 내부 아이피확인

다음 명령어로 nginx 정상 작동 여부 확인
curl -X GET http://{hostname}

서비스확인
kubectl get services

도커와 마찬가지로 호스트 PC에서 접속 가능하도록 포트와 type 노드 포트를 설정(각각의 노드에 포워딩 설정)
kubectl expose deployment nginx-deployment --port=80 --type=NodePort

최종
젠킨스 파일 패키징 -> Ansible 파일 이미지화 push && pull && hosts 전달 -> 쿠버네티스 pods 생성

1. ssh-copy-id [계정명]@hostIP 인증키 설정
2. ansible -i ./k8s/hosts kubernates -m ping -u [계정명] 테스트
3. yml에서 kubectl 위치를 못찾는경우 절대경로 설정 /usr/local/bin

advance jenkins)
pipeline script로 설정
