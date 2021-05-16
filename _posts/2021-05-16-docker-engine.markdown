---
layout: post
title: '[Docker] 도커 엔진'
subtitle: '도커 엔진'
categories: devops
tags: doker
comments: true
---

도커 엔진이 무엇인지 알아봅니다. 


## 도커 이미지와 컨테이너

- 도커 엔진에서 사용하는 기본 단위는 이미지와 컨테이너입니다.

## 도커 이미지

- 이미지는 컨테이너를 생성할 때 필요한 요소이며, 가상머신을 생성할 때 필요한 iso 파일과 비슷한 개념입니다.
- 도커에서 사용하는 이미지의 이름은 기본적으로 `[저장소 이름]/[이미지 이름]:[태그]` 형태로 구성됩니다.
- `[저장소 이름]` = 이미지가 저장된 장소. 저장소 이름이 명시되지 않으면 기본적으로 Docker Hub의 Ofiical 이미지를 받아옵니다.
- `[이미지 이름]` = 해당 이미지가 어떤 역할을 하는지 나타냄. 생략할 수 없습니다.
- `[태그]` = 이미지의 버전을 나타내며 생략하면 latest로 인식합니다.

## 도커 컨테이너

- 도커 이미지로 컨테이너를 생성하면 해당 이미지의 목적에 맞는 파일이 들어 있는 파일 시스템과 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성되고, 이를 도커 컨테이너라고 부릅니다. 

### 컨테이너 생성 
- `docker run -i -t ubuntu:14.04`
- 입력시 local에 ubuntu:14.04가 없기 때문에 Docker Hub에서 자동으로 이미지를 내려받습니다.
![Capture](/assets/img/post/docker/2021-5-16-docker-1.png)
- 컨테이너와 호스트의 파일시스템은 서로 독립적이므로 ls 명령어로 확인해볼 때 아무것도 설치되지 않음을 알 수 있습니다.

![Capture](/assets/img/post/docker/2021-5-16-docker-2.png)
- 다시 호스트의 도커 환경으로 돌아가고 싶으면, `exit`을 입력해주면 됩니다.
- 컨테이너를 종료하지 않고 호스트의 도커 환경으로 돌아가고 싶으면 `Ctrl + P, Q`를 입력합니다. 

- 이번에는 CentOS 이미지를 내려받겠습니다.
- `docker pull centos`
![Capture](/assets/img/post/docker/2021-5-16-docker-3.png)

- 이 CentOS 이미지로 컨테이너를 생성하려고 합니다. 
- `docker create -i -t --name mycentos centos:latest`
![Capture](/assets/img/post/docker/2021-5-16-docker-4.png)


- `docker run` 과는 달리 컨테이너 내부로 들어가지 않습니다. 
![Capture](/assets/img/post/docker/2021-5-16-docker-5.png)

- `docker run`과 `docker create`는 아래와 같은 차이가 있습니다. 
![Capture](/assets/img/post/docker/2021-5-16-docker-6.JPG)

- 따라서 `docker create` 후에는 `docker start`와 `docker attach` 명령어를 사용해서 컨테이너를 시작하고 내부로 들어가야합니다. 
![Capture](/assets/img/post/docker/2021-5-16-docker-7.png)

### 컨테이너 삭제
- 도커 컨테이너를 삭제하려면 `docker rm [컨테이너 이름]`을 입력합니다. 
![Capture](/assets/img/post/docker/2021-5-16-docker-8.png)
- 실행 중인 컨테이너는 삭제할 수 없어서 `docker stop`을 사용한 후에 삭제를 진행합니다. 

### 컨테이너를 외부에 노출
- 컨테이너는 가상 IP 주소를 할당받습니다. 
![Capture](/assets/img/post/docker/2021-5-16-docker-9.png)
- 아무런 설정을 하지 않았다면, 이 컨테이너는 외부에 접근할 수 없으며 도커 호스트에서만 접근할 수 있습니다.
- 외부에 컨테이너를 노출하기 위해서는 eth0의 IP와 포트를 호스트의 IP와 포트에 바인딩 해야합니다. 

- `docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04`를 사용하여 호스트의 80번 포트와 컨테이너의 80번 포트를 연결합니다. [호스트 포트]:[컨테이너 포트] 입니다.
![Capture](/assets/img/post/docker/2021-5-16-docker-10.png)
![Capture](/assets/img/post/docker/2021-5-16-docker-11.png)
![Capture](/assets/img/post/docker/2021-5-16-docker-12.png)
- 도커 엔진 호스트의 IP 주소:80 으로 아파치 웹서버에 접근할 수 있습니다.

### 컨테이너 애플리케이션 구축
- Ex) 데이터베이스와 웹서버 컨테이너를 연동
- `docker run -d --name wordpressdb -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wordpress mysql:5.7` => mysql 이미지를 이용해 데이터베이스 컨테이너 생성
- `docker run -d -e WORDPRESS_DB_PASSWORD=password --name wordpress --link wordpressdb:mysql -p 80 wordpress` => wordpress 이미지를 이용해 웹 서버 컨테이너 생성
![Capture](/assets/img/post/docker/2021-5-16-docker-13.png)
- 호스트의 52617번 포트와 연결됨 => 호스트의 IP외 52617번 포트로 wordpress 웹 서버에 접근할 수 있습니다. 
![Capture](/assets/img/post/docker/2021-5-16-docker-14.png)
- DB에러가 발생했는데, 이유를 모르겠네요. 일단 PASS...


## 도커 이미지
- 도커 허브에 있는 이미지를 검색하는 방법 = `docker search`
![Capture](/assets/img/post/docker/2021-5-16-docker-15.png)
- star가 많은 이미지를 사용하는 것을 추천합니다.
- 도커 이미지를 추가하는 방법은 크게 세 가지가 있습니다. 먼저 pull을 사용해 미리 만들어져있는 이미지를 가져오는 방법입니다. 두번째 방법은 컨테이너의 변경사항으로부터 이미지를 만드는 방법입니다. 세번째 방법은 Dockerfile을 빌드하는 방법입니다. 
- Dockerfile은 컨테이너에 설치하는 패키지, 추가해야하는 소스코드, 실행하는 명령어와 쉘 스크립트를 기록하는 파일입니다. 

### Dockerfile 작성
```bash
    mkdir dockerfile && cd dockerfile
    vi Dockerfile 
    FROM ubuntu:bionic
    RUN apt-get update
    RUN apt-get install -y git
``` 
- FROM은 어떤 이미지로부터 이미지를 생성할지 지정합니다.
- RUN은 명령어를 실행하라는 의미입니다.
- 먼저 `apt-get update`를 실행하고, 다음으로 `apt-get install -y git`을 실행합니다.

### Dockerfile 빌드
- `docker build -t ubuntu:dockerfile .`
![Capture](/assets/img/post/docker/2021-5-16-docker-16.png)

### 모니위키(moniwiki) 도커 파일 작성하기
```bash
    git clone https://github.com/nacyot/docker-moniwiki.git
    cd docker-moniwiki/moniwiki
```
- Dockerfile을 살펴보겠습니다.
```bash
    FROM ubuntu:14.04

    RUN apt-get update &&\
    apt-get -qq -y install git curl build-essential apache2 php5 libapache2-mod-php5 rcs

    WORKDIR /tmp
    RUN \
        curl -L -O https://github.com/wkpark/moniwiki/archive/v1.2.5p1.tar.gz &&\
        tar xf /tmp/v1.2.5p1.tar.gz &&\
        mv moniwiki-1.2.5p1 /var/www/html/moniwiki &&\
        chown -R www-data:www-data /var/www/html/moniwiki &&\
        chmod 777 /var/www/html/moniwiki/data/ /var/www/html/moniwiki/ &&\
        chmod +x /var/www/html/moniwiki/secure.sh &&\
        /var/www/html/moniwiki/secure.sh

    RUN a2enmod rewrite

    ENV APACHE_RUN_USER www-data
    ENV APACHE_RUN_GROUP www-data
    ENV APACHE_LOG_DIR /var/log/apache2

    EXPOSE 80

    CMD bash -c "source /etc/apache2/envvars && /usr/sbin/apache2 -D FOREGROUND"
```
- Dockerfile의 한 줄, 한 줄은 레이어라는 형태로 저장되기 때문에 RUN을 줄이면 레이어가 줄어들고, 캐시도 효율적으로 관리할 수 있습니다. 여기서 &&은 여러 명령어를 이어서 실행하기 위한 연산자이고, \은 명령어를 여러줄에 작성하기 위한 문자입니다.
- WORKDIR은 이후에 실행되는 모든 작업의 실행 디렉토리를 변경합니다.
- ENV는 컨테이너 실행 환경에 적용되는 환경변수의 기본값을 지정하는 지시자입니다.
- EXPOSE는 가상머신에 오픈할 포트를 지정해줍니다. 마지막 줄의 CMD에는 컨테이너에서 실행될 명령어를 지정해줍니다.

- `docker build -t nacyot/moniwiki:latest .`로 Dockerfile을 빌드
![Capture](/assets/img/post/docker/2021-5-16-docker-17.png)

- 모니위키 실행 `docker run -d -p 9999:80 nacyot/moniwiki:latest`
- http://127.0.0.1:9999/moniwiki/monisetup.php 로 들어가서 웹 페이지 확인



참고자료 
1. https://www.44bits.io/ko/post/easy-deploy-with-docker
2. 용찬호(2020), 시작하세요 도커/쿠버네티스, 
