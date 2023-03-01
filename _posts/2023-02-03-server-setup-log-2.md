---
title: "Server Setup Log 2"
excerpt: "home server setup log"

date: 2023-02-03
last_modified_at: 2023-02-03

categories:
  - server

tags:
  - server
  - linux

comments: true

toc: true
toc_sticky: true
---

## Intro

![server_room_giphy.gif](https://media.giphy.com/media/Vz3K2SHhpdgNq/giphy.gif){: .align-center}

↪ 지난 번 인프라 세팅부터 SSH 보안 설정까지 조금 더 local한 세팅을 완료했다고 한다면, 이번 포스트에서 기록할 내용은 web과 좀 더 가까운 내용들입니다. 살면서 한 번도 해보지 않았던 부분들이라 진행하는 내내 흥미진진한 기분으로 작업을 할 수 있었습니다. 실 서버를 드래곤볼하기 전부터 반복적으로 검색하던 내용이기도 했죠. 기대가 너무 많이 돼서요!

---

## Be prepared with Docker
### Docker

![docker_logo_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/docker_logo_01.jpg){: .align-center}

↪ 서버에서 여러가지 서비스들을 띄우기 위해 선택한 기술은 Docker였습니다. 되도록이면 컨테이너 기술을 써서 편리하게 서비스들을 띄우고 입맛에 맞게 컨트롤하면서도, 레퍼런스가 풍부한 기술을 우선해 고민했습니다. reddit이나 서버포럼, 나스당, dc 등등 각지의 커뮤니티에서 많이 목격되는 Proxmox나 헤놀로지 같은 OS도 고민의 대상이었습니다. 사실 이 부분은 1편에서 적는 것이 서순상 적당했을 것으로 보이긴 합니다.

결국 Ubuntu와 Docker 환경을 선택한 이유는

1. 레퍼런스의 든든함
2. 서비스할 계획이었던 이미지들의 존재 유무
3. 고래가 귀여움(...)

정도였습니다.

헤놀로지의 경우, 제가 원하는 정도의 확장성을 보장하지 못할 수도 있다는 걱정이 들었고 Proxmox의 경우에는 강력한 백업이나 다양한 OS를 사용할 수 있다는 장점이 매력으로 다가왔지만 동시에 저에게는 오버킬이면서 새로운 리서치가 우선되어야 한다는 부담이 있어 Ubuntu + Docker 형태의 서버를 구성하게 되었습니다.

docker 설치와 같은 경우에는, 공식 문서나 워낙 잘 되어 있기 때문에 따로 이곳에 기록하지는 않으려고 합니다.

[Install Docker On Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

위 문서를 그대로 참고에 설치 후, 따로 사용자 계정에 권한을 주지 않고 `sudo` command를 통해 docker cli를 사용 중입니다.

### Portainer

![portainer_logo_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/portainer_logo_01.jpg){: .align-center}

↪ 가장 먼저 필수적으로 올린 컨테이너는 Portainer입니다. 모든 컨테이너를 docker compose와 env 파일로 저장해놓고 GitHub를 통해 땡겨쓰는 저에게는 Terminal에서 cli로 컨테이너들을 관리하는 것보다 Portainer 내의 Stack으로 관리하는 편이 훨씬 편하게 다가왔습니다.

```yaml
version: "3.8"

services:
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    restart: unless-stopped
    ports:
      - ${PORTAINER_PORT}:9000
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - data:/data

volumes:
  data:
```

위와 같은 `docker-compose.yml` 로 Portainer를 띄웁니다. 관리자 계정을 설정해주고, local 머신을 선택해 간단히 ip를 지정해주고 나면 앞으로 생성할 docker container들을 Stack 메뉴에서 관리할 수 있습니다.

![portainer_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/portainer_01.jpg){: .align-center}

짜잔

### Nginx Proxy Manager

![nginx_proxy_manager_logo_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/nginx_proxy_manager_logo_01.jpg){: .align-center}

↪ 여러가지 서비스를 컨테이너로 띄워놓고 서브도메인으로 접근할 생각으로 homelab 구상을 시작했기 때문에 reverse proxy를 GUI 상으로 편리하게 관리하기 위해 NPM(Nginx Proxy Manager) 컨테이너를 띄워놓고 설정하게 되었습니다.

reverse proxy 설정 뿐만 아니라 SSL 설정 또한 편리하게 할 수 있기 때문에 NPM을 선택하게 되었는데, 짧은 기간이나마 사용해본 결과, 선택이 주효했던 것 같습니다.

NPM의 설정은 대부분 [서버포럼](https://svrforum.com/svr)의 [달소](https://blog.dalso.org/)님을 통해 손쉽게 구성할 수 있었습니다.

---

## Network
### Domain & DNS

![cloudflare_logo_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/cloudflare_logo_01.jpg){: .align-center}

↪ 도메인 구입을 고려하면서, 사실 처음에는 **구글 도메인**을 이용했습니다. 구글의 네임밸류를 너무 신뢰해버린 탓에... 그런 결정을 내렸지만 SSL 와일드카드 인증서를 설정하면서 곧바로 후회하게 되었습니다.

NPM 상에서 Let's Encrypt 와일드카드 SSL 인증서를 발급 받으려면 API 토큰을 통해 인증을 하는 과정이 필수적인데, 구글 도메인은 API 토큰을 제공해주지 않았던 것이죠...

![sobbing_giphy.gif](https://media.giphy.com/media/l3vR4CdLInXOhr3rO/giphy.gif){: .align-center}

결국 이 과정에서 **Cloudflare**로 갈아타게 되었는데, 하필이면 도메인을 구매하고 5일이 지나버려서 구글 도메인은 환불을 받지 못했습니다. 혹시나 이 글을 보시는 분들 중에 와일드카드 SSL 인증서를 사용하겠다 하시는 분은 꼭(...) Cloudflare를 선택하시기를 추천합니다.

![cloudflare_dns_](/assets/images/posts/2023-02-03-server-setup-log-2/cloudflare_dns_01.jpg){: .align-center}

DNS 설정은 복잡한 부분 없이 쉽게 진행할 수 있습니다. 현재 서버의 IP와 구매한 도메인을 바인딩해주고, 서브도메인들을 메인도메인에 바인딩해주면 완료입니다.

Cloudflare에서는 이메일 라우팅도 제공하기 때문에, 내 도메인으로 이루어진 이메일을 원하는 분이라면 꼭 설정하고 사용하시기를 강추 드립니다. 멋있잖아요...ㅋㅋㅋㅋ

### DDNS

```bash
# install ddclient
$ sudo apt-get install ddclient
# stop daemon
$ sudo /etc/init.d/ddclient stop

# edit your configuration file
# /etc/ddclient.conf

# start daemon
$ sudo /etc/init.d/ddclient start
```

![ddclient_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/ddclient_01.jpg){: .align-center}

↪ 대부분 개인들의 homelab 환경은 ISP에서 고정 ip를 제공해주는 형태는 아닐 겁니다. 그렇다보니 유동 ip라고는 해도 서버를 다이렉트로 다룰 수 없는 상황에서 ip가 바뀌어버린다면 집에 가기 전까지 서버 생각에 다른 일은 손에 잡히지도 않겠죠.

그래서 DDNS 설정이 필요한데, ddclient를 선택하게 되었습니다. 가장 간단한 방법으로 보여 선택하게 되었는데 위처럼 서비스를 설치하고, 설정 파일에 적절한 설정값을 잡아둔 뒤에 서비스를 실행하면 됩니다.

이제 다음과 같은 내용을 crontab을 통해 재부팅 시 ddclient 서비스가 실행되도록 스케줄링을 해주겠습니다.

```bash
# edit crontab list
$ sudo crantab -e

# add this line bottom
# @reboot root ddclient -daemon 1800 -syslog
```

### SSL

![cloudflare_api_token_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/cloudflare_api_token_01.jpg){: .align-center}

↪ SSL 인증서의 경우에도 그리 복잡한 과정은 불필요합니다. Nginx Proxy Manager가 정말 편하게 되어있으므로 SSL 와일드카드 인증서 발급을 위한 API 토큰만 Cloudflare에서 미리 발급 받아 복사해두면 됩니다.

![ssl_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/ssl_01.jpg){: .align-center}

↪ 이제 NPM 상에서 SSL을 발급받아주면 됩니다. SSL을 적용할 도메인 이름에는 와일드카드 인증서를 위한 `*.domainname.com`과 메인 도메인에서 리다이렉션을 시키기 위한 `domainname.com`을 둘 다 적어주는 것이 좋습니다. (ddclient의 설정에도 그래서 둘 다 적어주는 것이 좋습니다.)

조금 전에 복사해두었던 API 토큰은 바로 여기에 써주기 위한 것이었습니다. 이제 Save를 누르고 잠시 기다리기만 하면 완료입니다.

### Reverse Proxy

![reverse_proxy_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/reverse_proxy_01.jpg){: .align-center}

↪ 이제 Docker를 통해 올린 서비스들을 서브도메인과 매핑해주면 끝입니다! `Domain Names`에는 Cloudflare에 등록해둔 서브도메인 URL을 그대로 적어주고, `Schema`, `Forward Hostname / IP`, `Forward Port`에는 **NPM**의 입장에서 바라볼 서비스의 경로를 지정해주면 됩니다.

제 경우에는 현재 같은 호스트에서 돌아가고 있는 서비스들이기 때문에, NPM의 호스트인 서버 내부 IP를 통해 접근하도록 설정해주었습니다. 포트는 서비스 별로 컨테이너의 내부 포트와 매핑해준 호스트 포트를 적어주면 됩니다.

![reverse_proxy_02.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/reverse_proxy_02.jpg){: .align-center}

SSL도 바로 발급해두었던 인증서를 선택만 해주면 됩니다. 다음 옵션들은 상황에 맞게 설정해주었습니다.

- `Force SSL`: 모든 트래픽을 HTTPS 프로토콜을 사용하도록 리다이렉션을 강제합니다. 중간자 공격을 방지합니다.
- `HTTP/2 Support`: HTTP/2 버전을 지원하도록 해, 성능 상 이점을 얻을 수 있도록 합니다.
- `HSTS Enabled`: HTTP Strict Transport Security (HSTS)는 HTTPS 프로토콜로만 접속할 수 있도록 강제하는 옵션입니다. SSL stripping 공격을 방지할 수 있습니다.
- `HSTS Subdomains`: HSTS 옵션들 메인 도메인 뿐 아니라 서브도메인에도 모두 적용합니다. (해당 옵션은 제 경우에는 불필요했습니다.)

---

## Prevent being POTATO
### Suspend

![suspend_giphy.gif](https://media.giphy.com/media/HitAab11PjQZO/giphy.gif){: .align-center}

↪ 처음 서버를 세팅하고 나서 몇일동안 빈번하게 원격지에서 접속이 안 되는 기현상이 발생했습니다. 딱히 서버가 크래시가 나거나 전원이 나간 것도 아니었는데 무슨 문제인지 한참 서치를 해봤습니다. 결론은 그냥 일정 시간동안 입력이 없어 대기모드로 전환되었다는 것이었습니다. 아니 Ubuntu 'Server'인데 대기모드가 디폴트라니...

```bash
# check status
# Loaded: loaded
$ sudo systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target
# disable suspend
$ sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
# check status again
# Loaded: masked
$ sudo systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target
```

위와 같은 command로 대기 모드로 진입하는 것을 막아주는 설정을 해주고 재부팅을 합니다. 이후로는 대기모드에 들어가지 않고 계속 idle 상태로 잘 사용 중입니다.

### Soft Lockup

![soft_lockup_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/soft_lockup_01.jpg){: .align-center}

↪ 그렇게 또 몇일간 행복한 서버 운영을 하던 중... 돌연 서버로부터 응답이 없습니다. 이번엔 또 무슨 문젠가 깊은 인내의 시간을 보내고 집에 돌아와 서치를 반복했습니다. 서버에서 띄운 로그에는 `kernel:NMI watchdog: BUG: soft lockup - CPU#3 stuck for 1332s!` 이렇게 생겨먹은 서버의 단말마가 잔뜩 도배되어 있었습니다.

서치 결과, 가장 의심되는 항목은 그래픽카드와 Ubuntu OS 내장 nouveau 드라이버의 충돌이었습니다. 그래서 일단 nouveau 드라이버를 비활성화하고, nvidia driver를 설치해본 상태입니다. 실제로 이 원인 때문에 발생한 에러라 다음 치료제가 주효할지는 좀 더 긴 시간을 두고 지켜봐야 할 것 같습니다.

```bash
# update & upgrade before install
$ sudo apt update && sudo apt upgrade -y
# install nvidia-driver
$ sudo apt install -y ubuntu-drivers-common
# check nvidia driver list
$ ubuntu-drivers devices
```

nvidia-driver를 설치하고 설치 가능한 드라이버 리스트를 확인했습니다. 이 중에서 `recommended`가 붙어있는 드라이버를 설치했습니다.

```bash
# install recommended driver
$ sudo apt-get install nvidia-driver-525
```

우려와는 달리 설치 중 시간은 좀 걸렸지만, 이슈가 발생하지는 않았습니다. 이제, 기존에 동작하고 있던 nouveau 드라이버를 비활성화 시켜줘야 했습니다.

```bash
# make a new blacklist to disable the driver
# paste these two lines to the conf file
# 
# blacklist nouveau
# options nouveau modeset=0
$ sudo vim /etc/modprobe.d/blacklist-nouveau.conf
# apply configuration
$ sudo update-initramfs -u
# reboot
$ sudo reboot

# check graphic driver installed
$ nvidia-smi
```

---

## Epilogue

![flame_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/flame_01.jpg){: .align-center}

![plex_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/plex_01.jpg){: .align-center}

![komga_01.jpg.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/komga_01.jpg){: .align-center}

![kavita_01.jpg.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/kavita_01.jpg){: .align-center}

![nextcloud_01.jpg.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/nextcloud_01.jpg){: .align-center}

![zomboid_01.jpg.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/zomboid_01.jpg){: .align-center}

![zomboid_02.jpg.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/zomboid_02.jpg){: .align-center}



↪ 서버 원격 관리를 편하게 하기 위한 서비스들도 몇가지 사용 중이고 미디어 서버로 활용하기 위한 서비스들도 있고, 위의 서버 대시보드에서는 보이지 않지만 dedicated 게임 서버로도 활용 중입니다. 물론 토이 프로젝트의 DB 서버나 CICD 서버, 이미지 서버 등으로의 활용도 하는 중입니다.

앞으로도 authelia와 같은 서버 관리용 서비스나, qbittorrent, tandoor, piwigo, wireguard 같이 서버에서 활용하기 좋은 서비스들을 관심이 생기는대로 컨테이너로 띄워 여러 방법으로 활용해볼 생각입니다.

reddit이나 서버포럼 같은 곳들에서 보이는 서버 랙이나 스위치 같은 장비도 언젠가는 쓰게 될지도 모르겠네요. 당장은 서버를 사용하며 겪었던 soft lockup 이슈를 겪으면서, 하드웨어 watchdog 장비를 구해 WOL 설정과 함께 사용해보는걸 다음 목표로 생각하고 있습니다.

![server_01.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/server_01.jpg){: .align-center}
![server_02.jpg](/assets/images/posts/2023-02-03-server-setup-log-2/server_02.jpg){: .align-center}

---
