---
layout: post
title: "[docker] Confluence 구축하기"
description: >
  
tags: [devlog]
author: hdmun
comments: true
---

도커로 **Confluence**를 구축하는 방법을 정리해보았다.  


{:message}
  > 참고 ('도커 컨플루언스'로 검색하면 나오는 것들)  
  [개인 목적으로 Confluence 구축하기 (with Docker)](https://blog.lulab.net/alm/install-personal-confluence-with-docker/)  
  [atlassian jira, conflunce를 docker를 이용하여 설치](http://jmjeong.com/install-atlassian-software-in-docker/)  


**Confluence** 공식 Docker Hub에 설명이 잘 되어있어서 참고하였다. [Docker Hub confluence-server](https://hub.docker.com/r/atlassian/confluence-server/)  


{:message}
  > 구축한 시스템 정보  
  OS: Windows10 64bit  
  Docker Engine  
    Version 18.03.1-ce-win65 (17513)  
    OS/Arch: linux/amd64  


### Quick Start  

아래 명령을 실행하면 바로 컨테이너가 실행이 된다.  
~~~powershell
# in PowerShell
docker run -d `
  -v /c/users/hungr/docker-volume/confluence:/var/atlassian/application-data/confluence `
  --name="confluence" `
  -p 8090:8090 -p 8091:8091 `
  atlassian/confluence-server:6.6.6
~~~

페이지 접속  [http://localhost:8090](http://localhost:8090)  

구동 후 바로 접속하면 초기 설정을 해야한다.  
설정할 때 **Confluence** 라이센스 키가 필요하다. 없으면 구매하거나 체험판을 받아놓자.  [Atlassian Evaluation 라이센스 받기](https://my.atlassian.com/license/evaluation)  

**Confluence** 는 Embedded DB를 사용할 수 있지만  
도커를 사용하는 김에 DB 컨테이너를 추가해주자.  


일단 Quick Start 명령어로 실행한 **Confluence** 컨테이너를 지워준다.  
~~~ps1
docker stop confluence
docker rm confluence
~~~


볼륨에 연결했던 호스트 폴더의 데이터도 깔끔하게 지워주자  
/c/users/hungr/docker-volume/confluence  


### DB 컨테이너 추가 (PostgreSQL, Docker Compose 사용)  

DB 종류는 **PostgreSQL**로 작업하였다. (안 써봐서 이 기회에 써볼려고 선택)  

DB 컨테이너를 추가하려면 **Confluence** 컨테이너에 구동시 link 옵션을 추가해주어야 한다.  
컨테이너 두 개를 명령어로 관리해주기 귀찮아  
**docker compose**를 사용하여 컨테이너를 구동하였다.  

docker-compose.yml 파일에 아래와 같이 설정한다.  

~~~yml
# docker-compose.yml
version: '2.1'
services:
    confluence:
        image: atlassian/confluence-server:6.6.6
        restart: always
        container_name: confluence
        # environment:
        ports:
            - "8090:8090"
            - "8091:8091"
        volumes:
            - C:\Users\hungr\docker-volume\confluence-data:/var/atlassian/application-data/confluence
        links:
            - postgresql

    postgresql:
        image: postgres:10.4
        restart: always
        container_name: confluence-postgres
        environment:
            POSTGRES_DB: confluence  # 필수
            POSTGRES_USER: dbuser
            POSTGRES_PASSWORD: dbpassword
            POSTGRES_INITDB_ARGS: --encoding=UTF-8
            # PGDATA: /var/lib/postgresql/data
        ports:
            - "5432:5432"
        volumes:
            - postgres_data:/var/lib/postgresql/data

volumes:
    postgres_data:
~~~


**NOTE**: /var/lib/postgresql/data 경로를 Host 경로로 바로 마운트 시키면 윈도우에서 권한 문제가 발생해 **postgresql** 컨테이너가 실행되지 않는 문제가 발생한다. 권한 문제를 해결하기 위해 **postgres_data** 볼륨을 만들어 마운트 시켰다.  


아래 명령어를 실행  
~~~powershell
docker-compose -f docker-compose.yml up -d
~~~

페이지 접속 [http://localhost:8090](http://localhost:8090)  


**Confluence** 세팅은 의외로 간단해서 패스한다.  


{:message}
  > **TODO**
  1. Confluence Theme 적용하기  
  2. 볼륨 컨테이너 적용하기  
