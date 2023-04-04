# 개요

- https://docs.docker.com/engine/reference/builder/
- 단계별로 진행되는 인프라 구성 정보를 가진 파일
- 텍스트 형식
- 확장자 필요 없음(있어도 된다)
  - 공식 파일명 : Dockerfile(권장)
- 도커 파일을 n개를 모아서 하나의 환경을 구성하는 것 => 컴포즈
  - https://github.com/docker/awesome-compose

# 주요 명령어

<img src="https://firebasestorage.googleapis.com/v0/b/repo-27c12.appspot.com/o/docker%2Fdocker_file_option.png?alt=media&token=13987e2d-8aa6-47f9-858f-f78332f63d25">

- 대문자로 통일 권장(대소문자 관계 없음)
- 주석 
```
  #  주석
```

# lec_1 : FROM  : 베이스 이미지 기술

- 기술방법
  ```
  FROM 이미지명
  FROM 이미지명:태그
  FROM 이미지명@다이제스트값
  ```
- 도커 파일을 통해서 만드어지는 1개의 이미지의 베이스가 되는 이미지를 기술(출처)

- 실습
  - Dockerfile
  ```
  FROM ubuntu
  or
  # 베이스 이미지
  FROM ubuntu@sha256:f4270eef6cf86135dab38e64fe05708ff6983bff7282e2ba5c5d564603622bfe
  ```

- 이미지 생성
  - docker build -t test:1.0 ./lec_1
    - -t 이미지명:태그명
    - Dockerfile이 존재하는 경로를 지정
      - 만약 현재위치에 있다면 => . 
    ```
       => [internal] load build definition from Dockerfile                                                                0.1s
 => => transferring dockerfile: 71B                                                                                 0.0s
 => [internal] load .dockerignore                                                                                   0.0s
 => => transferring context: 2B                                                                                     0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                    0.0s
 => CACHED [1/1] FROM docker.io/library/ubuntu                                                                      0.0s
 => exporting to image                                                                                              0.0s
 => => exporting layers                                                                                             0.0s
 => => writing image sha256:f4270eef6cf86135dab38e64fe05708ff6983bff7282e2ba5c5d564603622bfe                        0.0s
    ```
    - docker build -t test:2.0 -f ./lec_1/Dockerfile.dig ./lec_1
    - 다이제스트 값은 도커 허브에서 공식 값 확인후 사용(권장)
      - 로컬이 이미지에서 확인
      - docker image ls --digests 이미지명
        - 여기서 나온값 사용

# lec_2 : ADD, COPY : Dockerfile 내에 다양한 파일 추가하는 명령어



- 이미지가 생성되거나, 이미지를 기반으로 컨테이너가 만들어질때 추가될 파일들을 기술한다
- 제외할 파일을 기술할수도 있다
  - .dockerignore
  ```
    # 제외한 파일
    po*.txt
  ```
  - ADD
    - 파일, URL상에 존재하는 파일, 압축파일(tar,gzip등)등을 원하는 위치에 저장
    - URL -> 다운로드해서 위치
    - 압축파일 -> 해제해서 위치
    
  - COPY
    - 파일만 카피
    - 필요하면 저장위치에 파일명 지정
    - URL, 압축해제 X

- Dockerfile

  ```
  # 베이스 이미지
  FROM ubuntu

  # 압축해제후 추가
  ADD web.tar /home/

  # URL 다운로드 후 추가
  ADD https://raw.githubusercontent.com/ultralytics/yolov5/master/requirements.txt /home/requirements.txt

  # 파일 추가, 이름을 바꾸서 넣어보기
  ADD readme.md /home/readme2.md

  # 특정 이름으로 시작하는 모든 파일(패턴지정) 추가
  ADD po*.txt /home/

  # 특정 이름+한글자로 시작하는 모든 파일 추가
  ADD to?.txt /home/

  ```

- step 1 이미지 생성 test:1.1
  -  docker build -t test:1.1 ./lec_2
  
  ```
    => [internal] load build definition from Dockerfile                                                                0.1s
  => => transferring dockerfile: 525B                                                                                0.0s
  => [internal] load .dockerignore                                                                                   0.1s
  => => transferring context: 67B                                                                                    0.0s
  => [internal] load metadata for docker.io/library/ubuntu:latest                                                    0.0s
  => CACHED [1/6] FROM docker.io/library/ubuntu                                                                      0.0s
  => [internal] load build context                                                                                   0.1s
  => => transferring context: 161.41kB                                                                               0.0s
  => CACHED https://raw.githubusercontent.com/ultralytics/yolov5/master/requirements.txt                             0.0s
  => [2/6] ADD web.tar /home/                                                                                        0.7s
  => [3/6] ADD https://raw.githubusercontent.com/ultralytics/yolov5/master/requirements.txt /home/requirements.txt   0.2s
  => [4/6] ADD readme.md /home/readme2.md                                                                            0.1s
  => [5/6] ADD po*.txt /home/                                                                                        0.1s
  => [6/6] ADD to?.txt /home/                                                                                        0.1s
  => exporting to image                                                                                              0.3s
  => => exporting layers                                                                                             0.3s
  => => writing image sha256:99fcefdf004c2a488dd051076fa57e8f131f0fa832545fff62605ae89c67bcf4                        0.0s
  => => naming to docker.io/library/test:1.1                                                                         0.0s
  ```
- step 2 컨테이너 생성 t11
  - docker run -itd --name t11 test:1.1
- step 3 컨테이너의 bash 실행
  - docker exec -it t11 bash
- step 4 ls 명령으로 파일이 잘 추가되었는지 확인
  ```
    root@247532e6810a:/# ls /home
    readme2.md  requirements.txt  to1.txt  web
    root@247532e6810a:/# ls /home -al
    total 16
    drwxr-xr-x 1 root root 4096 Mar 30 04:55 .
    drwxr-xr-x 1 root root 4096 Mar 30 04:56 ..
    -rwxr-xr-x 1 root root    0 Mar 30 04:25 readme2.md
    -rw------- 1 root root 1593 Jan  1  1970 requirements.txt
    -rwxr-xr-x 1 root root    0 Mar 30 04:25 to1.txt
    drwxr-xr-x 3 root root 4096 Mar 30 04:29 web
    root@247532e6810a:/# ls /home/web -al
    total 16
    drwxr-xr-x 3 root root 4096 Mar 30 04:29 .
    drwxr-xr-x 1 root root 4096 Mar 30 04:55 ..
    drwxr-xr-x 2 root root 4096 Oct 26  2017 css
    -rwxr-xr-x 1 root root  716 Mar 30 04:29 index.html
    root@247532e6810a:/# ls /home/web/css -al
    total 164
    drwxr-xr-x 2 root root   4096 Oct 26  2017 .
    drwxr-xr-x 3 root root   4096 Mar 30 04:29 ..
    -rwxr-xr-x 1 root root 156885 Oct 26  2017 bootstrap.css
  ```

# lec_3 : RUN, CMD, COPY, EXPOSE 명령어 

- RUN
  - shell 형식으로 기술
  - /bin/bash 에서 실행한다
  - 표현방식
    - RUN echo "shell 형식"
    - RUN ["echo", "exec 형식"]
    - RUN ["/bin/bash", "-C", "ehco : 'exec에서 bash 사용' "]

- CMD
  - 데몬 실행
  - 이미지를 바탕으로 생성된 컨테이너 안에서 명령을 수행할때 사용
  - Dockerfile 내에서는 오직 한번만 기술할수 있다
  - 여러개를 작성해도, 마지막 1개만 유효하다
  - 기술
    - Exec 형식
      - CMD ["nginx", "-g", "daemon off;" ]
    - shell 형식
      - CMD nginx -g 'daemon off;'

- Dockerfile
  - FROM, RUN, ADD, COPY, CMD 를 연계해서
  - ubuntu 기반 nginx 서버 세팅 + 초기 페이지 소스(web.tar)를 홈디렉토리에 풀어서 컨테이너 가동하후 => 바로 웹페이지 접속 => 페이지 확인
  ```
    # 베이스 이미지
    FROM ubuntu

    # ubuntu os에서 nginx를 설치하기 위한 작업 -> 설치관리자 레포지토리 최신으로 갱신
    RUN apt-get -y update && apt-get -y upgrade
    # nginx 설치
    RUN apt-get -y install nginx

    # EXPOSE, 이 이미지를 기반으로 만들어질 컨테이너의 공개 포트 번호 지정
    # nginx 웹서버이므로 통상 80 포트를 사용
    EXPOSE 80

    # nginx를 기본으로 설치하면, 환경설정상 루트 디렉토리(웹이 인식하는 물리적 경로)
    # ->html을 기본적으로 어디다가 두는가? -> /var/www/html/*.html
    ADD web.tar /var/www/html/

    # COPY 연습
    COPY readme.txt /var/www/html/readme2.txt

    # 컨테이너가 만들어지고 실행되면 nginx가 가동
    CMD ["nginx", "-g", "daemon off;"]

  ```

  - step1 이미지 생성 my_nginx:1.0
    - docker build -t my_nginx:1.0 ./lec_3
  - step2 컨네이너 생성 my_ng10
    - docker run -itd -p 80:80 --name my_ng10 my_nginx:1.0
  - step3 http://127.0.0.1 접속
    - tar 파일 압축해제하면 web 폴더가 생겨서 문제됨
      -> 경로조정해서 tar를 재배포

# lec_4 ENTRYPOINT, WORKDIR, ENV 명령어

- ENTRYPOINT
  - CMD와 비교
    ```
      ENTRYPOINT ["nginx"]
      CMD ["-g","daemon off;"]
    ```
    - ENTRYPOINT는 명령 자체를 지정,  CMD 에서 명령 인수를 지정하는 방식으로 섞어서 사용 가능
  - CMD 특성
    - docker run 명령에서 특정한 명령을 지정한다면, CMD가 항상 우선순위로 수행된다

- WORKDIR
  - 작업 디렉토리
  - Dockerfile 내에서 작성한 명령들이 수행하는 작업디렉토리
  - 해당 디렉토리가 없다면, 생성
  - 영향을 받는 명령어
    - RUN, CMD, ENTRYPOINT, COPY, ADD
  - 여러번 사용 가능

- ENV
  - Dockerfile내에서 환경변수 설정용
  - 방식
    - ENV 키 값 => 한줄에 한개씩만 설정 가능
    - ENV 키=값 => 한줄에 여러개 설정 가능

- Dockerfile

  ```
    # 베이스 이미지
    FROM mariadb

    # 환경변수 -> 디비 , 계정, 루트 비번
    ENV MARIADB_USER "ai"
    ENV MARIADB_PASSWORD 1234
    ENV MARIADB_USER="ai1" MARIADB_PASSWORD="12345"
    ENV MARIADB_ROOT_PASSWORD 1234
  ```

- docker build -t env ./lec_4
- docker run -d -p 3308:3306 --name env-db env
- docker exec -it env-db bash

  ```
  root@9e52c0c7420a:/# mysql -u root -p
  Enter password:
  Welcome to the MariaDB monitor.  Commands end with ; or \g.
  Your MariaDB connection id is 3
  Server version: 10.11.2-MariaDB-1:10.11.2+maria~ubu2204 mariadb.org binary distribution

  Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  MariaDB [(none)]> exit
  Bye
  root@9e52c0c7420a:/# mysql -u ai -p
  Enter password:
  ERROR 1045 (28000): Access denied for user 'ai'@'localhost' (using password: YES)
  root@9e52c0c7420a:/# mysql -u ai2 -p
  Enter password:
  ERROR 1045 (28000): Access denied for user 'ai2'@'localhost' (using password: YES)
  root@9e52c0c7420a:/# exit
  exit
  ```

- Dockerfile.wd
  ```
  FROM ubuntu

  # 환경변수
  ENV SERVICE_ROOT /home
  ENV SERVICE_VENV venv
  ENV SERVICE_DIR web

  # 환경변수 조합해서 WORKDIR 지정
  # 해당 디렉토리가 없다면 만들어서 현재 디렉토리로 이동
  # mkdir, cd를 포함한 명령
  WORKDIR $SERVICE_ROOT/$SERVICE_VENV/$SERVICE_DIR

  # 실행
  RUN ["pwd"]

  ```

- docker build -t wd -f ./lec_4/Dockerfile.wd ./lec_4
- docker run -itd --name wd10 wd
- docker exec -it wd10 bash
  - 컨테이너에 진입하면 시작 디렉토리가 작업디렉토가 된다
  ```
  root@0a6936c11c94:/home/venv/web# exit
  exit
  ```


# lec_5 ONBUILD 명령어

- 이미지를 빌드하면 실행되는 명령어
- Dockerfile 
  ```
  # 베이스 이미지
  FROM ubuntu

  # 패키지업데이트, nginx 설치
  RUN apt-get -y update && apt-get -y upgrade
  RUN apt-get -y install nginx

  # 포트 설정
  EXPOSE 80

  # 빌드 완료후 이 이미지를 기반으로 새로운 이미지를 빌드하면 실행할 명령
  ONBUILD ADD web.tar /var/www/html

  # 컨테이너에서 실행할 명령
  CMD ["nginx", "-g", "daemon off;"]
  ```
- 이미지 빌드
  - docker build -t test:3.0 ./lec_5
- 컨테이너 생성
  - docker run -d -p 80:80 --name t30 test:3.0
- http://127.0.0.1 접속
  - Nginx ... 페이지 나옴
- 실제 nginx의 도큐먼트 루트 확인
  ```
  docker exec -it t30 bash
  root@73841c5460c4:/# ls /var/www/html
  index.nginx-debian.html
  root@73841c5460c4:/# exit
  exit
  ```

- 2차 Dockerfile 구성
  - Dockerfile.2 (파일명은 임의로 구성)
  - -f
  ```
  # 베이스 이미지를 내가 만든 커스터 이미지로 지정
  # 통상적으로는 한개의 Dockerfile 안에서 베이스구성 => 2차 이미지도 바로 구성
  FROM test:3.0
  ```
- 이미지 빌드
  - docker build -t test:3.1 -f ./lec_5/Dockerfile.2 ./lec_5
- 기존 80포트를 사용하고 있는 컨테이너 중지
  - docker stop t30
- 컨테이너 생성및 가동
  - docker run -d -p 80:80 --name t31 test:3.1
- http://127.0.0.1 접속
  - 커스텀 페이지 확인됨

# lec_6 : STOPSIGNAL, HEALTHCHECK



- STOPSIGNAL
  - 컨테이너 종료시 송신하는 시그널
- HEALTHCHECK
  - 컨테이너 안에서 프로세스가 정상적으로 작동하는지 체크
  - mariadb, flask가 프로세스가 현재 정상적으로 가동중인지 체크 -> 하자 -> 컨테이너를 교체(삭제,추가) or 복구
  - 인터벌 지정 (컴포즈에서 사용)
    - 반복시간 설정
    - 응답 반응 시간 설정 -> 응답 지연시간 -> 성능평가 사용



- Dockerfile
  ```
  FROM ubuntu

  # 30초 간격으로 특정 사이트 접속 체크, 
  # 응답 지연은 30초이내에 들어올때까지는 오류없음
  HEALTHCHECK --interval=30s --timeout=30s CMD [ "curl", "-f", "http://127.0.0.1", "||", "exit 1" ]

  # 디비 체크 예시
  # HEALTHCHECK --interval=30s --timeout=30s CMD ["mysqladmin", "ping", "-h", "127.0.0.1", "--password=12341234", "--silent" ]
  ```

- 이미지 빌드
  - docker build -t test:4.0 ./lec_6
- 컨테이너 생성,실행
  - docker run -itd --name t40 test:4.0
- 컨테이너 정보 확인
  - docker container inspect t40
  ```
    "Health": {
                  "Status": "unhealthy",
                  "FailingStreak": 5,
                  "Log": [
                      {
                          "Start": "2023-03-31T01:17:32.0407285Z",
                          "End": "2023-03-31T01:17:32.1658428Z",
                          "ExitCode": -1,
                          "Output": "OCI runtime exec failed: exec failed: unable to start container process: exec: \"curl\": executable file not found in $PATH: unknown"
                      },
                      ...
  ```
  - 실제 체크가 잘 되게 명령어 path 확인

# lec_7 : USER



- USER
  - 사용자를 지정한다
  - 영향을 받는 명령어
    - RUN, CMD, ENTRYPOINT

- Dockerfile
  ```
    FROM ubuntu

    RUN ["adduser", "ai"]
    RUN ["whoami"]

    # 기존사용자(root) -> 현재 사용자를 ai로 변경
    USER ai
    RUN ["whoami"]
  ```

- docker build -t test:5.0 ./lec_7
- docker run -itd --name t50 test:5.0
- docker exec -it t50 bash
  ```
    ai@f3593ed77c82:/$
  ```

# lec_8 : LABEL, ARG 

- LABEL
  - 이미지의 정보
  - 버전, 작성자, 코멘트 등등
  - LABEL 키 값 
  - LABEL 키=값
- ARG
  - Dockerfile 안에서 사용하는 변수
  - 정의
    - ARG 변수명=값
  - 사용
    - $변수명


- Dockerfile
  ```
    FROM ubuntu

    # 도커파일 내부에서 사용하는 변수
    ARG NAME="AI_KIM"

    # 이미지상에 기록한 정보 -> inspect
    LABEL author=$NAME
    LABEL title "우분트 컨테이너"
    LABEL version="1.0"
    LABEL description="Good"

    RUN echo $NAME
  ```

- docker build -t test:6.0 ./lec_8
- docker image inspect --format="{{ .Config.Labels }}"  test:6.0
  ```
    map[author:AI_KIM description:Good org.opencontainers.image.ref.name:ubuntu org.opencontainers.image.version:22.04 title:우분트 컨테이너 version:1.0]
  ```
  - 2개의 기본정보 + 커스텀 추가 정보

# lec_9 : SHELL

- SHELL
  - 기본 쉘을 지정
  - 지정하지 않으면
    - 리눅스 : /bin/bash
  - 배시 이외의 쉘을 지정하고 싶다면 사용

- Dockerfile
  ```
    FROM ubuntu

    # 기본쉘 지정
    SHELL [ "/bin/bash", "-c" ]

    RUN echo "hi"
  ```

# lec_10 : VOLUME

- VOLUME
  - 이미지에 볼륨 할당
  - [ "이미지내경로(컨테이너생성시 위치고려)","이미지내경로(컨테이너생성시 위치고려)", ..  ]
  - 나열시
    - 호스트 OS상에 볼륨이 잡힌다
    - 볼륨 마운트 방식 지원

- Dockerfile
  ```
    FROM centos

    VOLUME [ "/var/log/" , "/data/" ]
  ```

- docker build -t test:7.0 ./lec_10
-  docker run -itd --name t70 test:7.0    
  ```             83e39f613371daa74f60d89327e3d78e329f2f2e932ffa3a27e67cae1c0c0b32
  ```
-  docker exec -it t70 bash

  ```
    [root@83e39f613371 /]# ls /var/log
    anaconda  btmp  lastlog  private  wtmp
    [root@83e39f613371 /]# ls /data
    [root@83e39f613371 /]# ls
    bin   dev  home  lib64       media  opt   root  sbin  sys  usr
    data  etc  lib   lost+found  mnt    proc  run   srv   tmp  var
    [root@83e39f613371 /]# exit
    exit
  ```
- docker stop t70
- docker rm t70
- 해석
  - 도커 클라이언트상에 존재하는 볼륨 마운트는 컨테이너와 생애주기를 다르게 가져간다
  - 볼륨 지정과 동시에 해당 컨테이너 상의 특정 폴더에 있던 데이터는 그대로 도커 클라이언트상 볼륨에 저장된다
  - 컨테이너가 삭제되더라고, 해당 데이터는 유지된다

- Dockerfile
  ```
    FROM centos

    VOLUME [ "/var/log/" , "/data/" ]
  ```

# lec_11 : NETWORK : 없음

- Dockerfile
  ```
  FROM centos

  # NETWORK 명령어 없다 => compose에서 해결, 컨테이너 생성시 옵션명령으로 해결
  NETWORK
  ```