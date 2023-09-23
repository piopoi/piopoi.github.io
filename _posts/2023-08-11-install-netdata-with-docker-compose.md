---
title: "Install Netdata with docker-compose"
date: 2023-08-11
categories: 
  - Server
tags:
  - Linux/ubuntu22.04
  - Monitoring
  - Docker
toc: true
toc_sticky: true
#use_math: true
---
<br>

# 1. 서버 환경

개인 서버가 1대라서 Netdata를 서비스 할 서버와 모니터링할 노드가 같은 서버다.

- ubuntu 22.04.3
- docker, docker-compose
- netdata 설정파일을 저장할 디렉토리: /home/{사용자명}/docker/netdata

<br>

# 2. Netdata 설치 & 실행

## 2-1. volumes에 마운트 할 디렉토리 생성

netdata 데이터를 저장할 디렉토리로 이동한다. (없으면 생성: mkdir)
```bash
cd /home/{사용자명}/docker/netdata
```
<br>

디렉토리 3개(config, lib, cache)를 생성한다.
```bash
mkdir config lib cache
```
<br>

## 2-2. docker-compose 파일 작성

docker-compose.yml 파일을 생성하고,
```bash
vi docker-compose.yml
```
<br>

아래 내용을 저장한다.  
(volumes의 경로를 사용자에 맞게 수정해야 한다.)

```bash
version: '3'
services:
  netdata:
    image: netdata/netdata
    container_name: netdata
    pid: host
    network_mode: host
    restart: unless-stopped
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      - /home/{사용자명}/docker/netdata/config:/etc/netdata
      - /home/{사용자명}/docker/netdata/lib:/var/lib/netdata
      - /home/{사용자명}/docker/netdata/cache:/var/cache/netdata
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
```
<br>

# 2-3. docker container 생성

아래 명령어를 실행하면 도커 이미지를 가져와서 컨테이너까지 띄워준다.
```bash
sudo docker compose up -d
```
<br>

# 3. Netdata 접속

브라우저에서 아래 URL을 입력하여 Netdata 페이지에 접속한다.  
Netdata의 기본 Port는 **19999**이다.
```bash
http://{서버 IP}:19999
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2023/0923/230923_01.png" width="100%"/><br>

<br>

# 4. 후기
netdata를 먼저 사용해본 후 화면이 마음에 안들면 grafana를 추가로 사용 가능한지 알아보려고 했는데, netdata 단독 사용으로도 충분할 만큼 화면이 깔끔하다.

다만, **관리자 계정이나 접속 권한을 설정할 수 있는 옵션이 보이지 않는다.**  
원래 목적이 외부망에서 접속 가능한 모니터링 페이지를 구축하려고 한 것이라, 조금 더 고민해봐야 할 듯 싶다.

<br>
<br>

# References.
[https://learn.netdata.cloud/docs/installing/docker](https://learn.netdata.cloud/docs/installing/docker)