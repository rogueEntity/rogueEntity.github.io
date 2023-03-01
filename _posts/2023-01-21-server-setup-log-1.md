---
title: "Server Setup Log 1"
excerpt: "home server setup log"

date: 2023-01-21
last_modified_at: 2023-01-21

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

![potato_server_giphy.gif](https://media2.giphy.com/media/ijvngPcd8kNOha4Se1/giphy.gif){: .align-center}

↪ 원래부터 사용하던 Windows PC를 새롭게 빌딩하면서 남는 부품으로 서버를 빌딩했습니다. 기존에 쓰던 지역 케이블 대신 KT 인터넷도 계약해 지금까지 GCP와 AWS, WSL에서의 폐관수련으로 쌓은 홈 서버 성취를 극성으로 펼쳐볼 생각에 가슴이 두근거렸습니다. 가장 기본적으로 라우터 셋업부터 시작해 진행한 자취를 남겨놓으려고 합니다. 나중에라도 베어핸드부터 다시 시작할 일이 있으면 미래의 내가 지금의 이 글을 보고 감동의 눈물을 흘리기를 바라며 작성합니다.

---

## Things I had to decide
### Operating System

![ubuntu_kinetic_kudu_01.jpg](/assets/images/posts/2023-01-21-server-setup-log-1/ubuntu_kinetic_kudu_01.jpg){: .align-center}

↪ 가장 먼저 고민했던 부분은 OS를 결정하는 부분이었습니다. 리서치가 가장 편했으면 하고 홈 서버 용도도 범용성을 제일 크게 생각하고 있어서 Ubuntu를 사용할 생각은 예전부터 확고했습니다. 다만, 20.04 버전을 선택할지 22.04를 선택할지에 대해 고민하고 있었습니다.

홈 서버다 보니 혹시나 있을지 모르는 계획에 없던 호환성 문제 같은 경우 크게 걱정하지 않고 미래의 내가 감당할 것이란 희망찬 생각으로 큰 문제가 되지 않았습니다. 다만, 현재 진행 중인 토이 프로젝트의 CI/CD 세팅 중 Docker로 올려놓은 Jenkin 이미지 내에서 Docker 빌딩을 할 때 GLIBC 버전 문제가 있어 머릿속으로 엄청난 토론대회를 펼치는 일이 있었습니다.

이 문제는 사실 Docker Image를 커스텀해 내가 원하는 이미지로 Jenkins를 올리면 해결되는 문제지만, 나는 Docker를 내가 편하게 날먹하려고 쓰는건데 이걸 하는게 맞는건가? 라는 의문 때문에 쭉 고민이 되었던 것입니다. 그리고 사실 성향 자체가 up-to-date된 버전을 좋아하는 것도 있지만 가장 큰 문제는 20.04의 'Focal Fossa'보다 22.04의 'Jammy Jellyfish'가 멋있다는 사실이었습니다. (진짜로...)

그렇게 Ubuntu 버전 별 네이밍을 보면서('Hirsute Hippo' 로고를 보고 파안대소함) 모든 고민이 해소되었습니다. 바로바로 22.10 버전이 **'Kidetic Kudu'**라는 이름이고 로고가 차마 거부할 수 없이 멋있기 때문입니다.(저는 Ubuntu Server를 사용합니다.) 그래서 Jenkins는 열심히 써봤으니 다른 CI를 경험해보자는 취지를 억지로 밀어넣어서 차후 Dagger CI를 염두에 두기로 하고 바로 22.10으로 업그레이드를 진행했습니다.

---

## Network
### Home Network

![home_network_01.jpg](/assets/images/posts/2023-01-21-server-setup-log-1/home_network_01.jpg){: .align-center}

↪ 사실상 제 방에서만 단일 네트워크가 구성이 되어있습니다. 장비가 유난히 많은 것도 아니고 스위치 장비나 VLAN 구성을 해야할 이유도 딱히 없기도 하다보니 굉장히 심플한 구조의 홈 네트워크가 되었습니다.

뭔가... 서버 셋업이나 네트워크 구성보다도 가장 힘들고 시간이 오래 걸렸던 부분은 ISP 광랜 가설부터 서버 및 PC 빌딩과 케이블 타이와 벨크로 그리고 멀티탭으로 이루어진 뻘짓까지의 물리적인 셋업이었습니다. 좁은 책상 밑에 들어가서 뒷면과 벽 사이의 공간이 10cm 정도 되는 타공판에 작업을 하거나 ISP 광랜 설치 시 현재 거주하는 주택의 구조 때문에 KT 기사님과 함께 여기 저기 올라갔다 내려왔다를 반복하며 광섬유 덩어리를 던지고 받고 하는 일 같이 진짜 재밌는 일들이 있었습니다.

### Port Forward

![port_forward_01.jpg](/assets/images/posts/2023-01-21-server-setup-log-1/port_forward_01.jpg){: .align-center}

↪ 가장 먼저 세팅해 준 부분은 포트포워딩이었습니다. 차후에 Nginx Proxy Manager를 통해 리버스 프록시를 사용할 생각이었기 때문에, 서비스가 마다 포트를 열어줄 필요가 없었습니다. 그래서 SSH, HTTP, HTTPS와 DB 커넥션을 위한 포트 정도만 열어주도록 설정했습니다.

### WOL

![wol_01.jpg](/assets/images/posts/2023-01-21-server-setup-log-1/wol_01.jpg){: .align-center}

↪ 라우터 관리 페이지에 접속한 김에 Wake On Lan 기능도 설정해주었습니다. 과거 WSL2 상에서 dedicated game server를 운영하면서 WOL 덕을 몇번 보고 나니 설정해놓으면 언젠가는 꼭 한 번 쓰겠구나 싶어서 서버 뿐만 아니라 다른 기기들도 등록해주게 되었습니다.

퇴근길에 PC나 맥북을 미리 켜놓는 것만 생각해도 필수이긴 합니다.

---

## SSH

![terminal_giphy.gif](https://media.giphy.com/media/VF0WIRjfwvFERopBFY/giphy.gif){: .align-center}

↪ 가장 먼저 서버 셋업을 시작하면서 설정한 부분이 SSH 보안 설정이었습니다. 저는 보안에 대해 딱히 공부해본 이력이 없기 때문에 끊임없는 구글링을 통해 얻은 기초적이고 필수적인 보안 설정 구결을 적어보도록 하겠습니다.

1. SSH 포트 변경
2. 방화벽 설정
3. root login 차단
4. Designated User login 허용
5. password 대신 public key로 login
6. fail2ban
7. Session Timeout 설정
8. 2FA Auth 설정

크게는 위와 같은 보안 설정들이 가장 기본적으로 세팅해야 할 리스트로 정리가 되었습니다. 물론 이 중 상황에 따라, 취향에 따라 굳이 설정하지 않아도 좋은 경우가 있으니 제 세팅은 그냥 제 취향이라고 생각하면 됩니다.

### SSH 포트 변경

```bash
# ssh 설정 오픈
$ sudo vim /etc/ssh/sshd_config
```
```bash
# /etc/ssh/sshd_config
# 주석처리된 다음 프로퍼티의 값을 변경하고 주석 해제
Port 9999 # 포트 넘버는 원하는대로 설정
```

↪ ssh 기본 포트는 22번으로 외부에서 원치 않는 접속 시도를 최대한 회피하기 위해 well-known port인 22번을 피하는 것이 좋다고 합니다. 저도 포트포워딩을 먼저 진행하면서 생각해둔 번호로 설정을 해두었고, 이제 서버에서도 설정 값을 바꿔주었습니다.

### 방화벽 설정

```bash
# iptables Chain 조회
$ sudo iptables -L --line-numbers
```

↪ Ubuntu 22.10 기준 iptables로 조회해본 방화벽 기본 설정은 제약 사항이 없었습니다. 저의 경우, 공유기 상에서 지정한 포트 이외에는 전부 포트를 닫아놓았기 때문에 따로 방화벽을 설정하지 않고 넘어갔습니다.

### root login 차단

```bash
# ssh 설정 오픈
$ sudo vim /etc/ssh/sshd_config
```
```bash
# /etc/ssh/sshd_config
# 주석처리된 다음 프로퍼티의 값을 변경하고 주석 해제
PermitRootLogin no
```

↪ 기본 값은 `prohibit-password`로 되어 있습니다. 원격으로 root login을 차단할 계획이기 때문에 `no`로 설정해주었습니다.

### Designated User login 허용

```bash
# ssh 설정 오픈
$ sudo vim /etc/ssh/sshd_config
```
```bash
# /etc/ssh/sshd_config
# 다음 프로퍼티를 추가
AllowUsers use1,user2,user3
```

↪ 저의 경우에는 개인 홈 서버라 유저를 하나 밖에 만들지 않았습니다. ssh login을 허용할 유저명을 적어줍니다.

### fail2ban

↪ SSH 포트를 열어놓고 사용하다보면, Brute Force 공격이 발생하기 마련이라고 합니다. 아직까지 실제로 접속 시도가 발생해 불안해 본 적은 없지만 미리 조치를 취해놓으려고 합니다. 우선 fail2ban을 설치하고, 부팅 시 자동으로 시작되도록 설정합니다.

개인 설정이 필요하다면 다음과 같은 위치의 설정 파일에서 설정값을 지정한 뒤, fail2ban 서비스를 재구동시켜 적용해줍니다.

```bash
# fail2ban 설치
$ sudo apt install fail2ban
# 부팅 시 시작되도록 설정
$ sudo systemctl enable fail2ban
```

```bash
# fail2ban 개인 설정
$ sudo vim /etc/fail2ban/jail.local
# 재구동
$ sudo systemctl restart fail2ban
```

![fail2ban_01.jpg](/assets/images/posts/2023-01-21-server-setup-log-1/fail2ban_01.jpg){: .align-center}

다음과 같은 커맨드로 fail2ban을 활용할 수 있습니다.

```bash
# 상태 확인
$ fail2ban-client status
# fail2ban log
$ sudo tail -f /var/log/fail2ban.log
# 접속 시도 log
$ sudo tail -f /var/log/auth.log
# banned ip 해제
$ sudo fail2ban-client set sshd unbanip 111.222.333.444
```

### Session Timeout 설정

```bash
# 사용하는 쉘의 설정 오픈
$ vim ~/.zshrc
# 쉘 재기동
$ source ~/.zshrc
```
```bash
# Session Timeout
export TMOUT=1800
```

↪ 저는 zsh를 사용하기 때문에 zshrc 파일로 세션 타임아웃 설정을 해주었습니다. bash를 사용하는 경우에는 bashrc에서 설정 값을 잡아주면 됩니다.

### password 대신 public key로 login

```bash
# ed25519 알고리즘으로 공개키 인증키 쌍 생성
$ ssh-keygen -t ed25519
```

![ssh_keygen_01.jpg](/assets/images/posts/2023-01-21-server-setup-log-1/ssh_keygen_01.jpg){: .align-center}

↪ `ssh-keygen`을 통해 키 쌍을 생성하려고 하면, 다음과 같은 항목들을 물어봅니다.

1. 키를 생성해 파일로 저장할 경로
2. passphrase
3. 입력한 passphrase 확인

이 항목들을 입력하고 나면 지정한 경로에 키 쌍이 파일로 생성되는데 공개키는 서버 상에, 인증키는 개인적으로 보관해야 합니다. 이제 공개키를 인증에 사용하기 위해 적절한 경로에 위치시켜야 합니다.

```bash
# 아래 위치에 공개키 입력
$ sudo vim ~/.ssh/authorized_keys
# 다음과 같이 작업할 수 있음
$ cat id_ed25519.pub >> ~/.ssh/authorized_keys
```

이제 서버의 ssh에 비밀번호 로그인이 불가능하도록 설정을 변경해주어야 합니다.

```bash
# ssh 설정 오픈
$ sudo vim /etc/ssh/sshd_config
```
```bash
# /etc/ssh/sshd_config
PasswordAuthentication no
```

### 2FA Auth 설정

↪ 마지막으로 2FA 인증을 설정해 로그인 시 OTP 인증이 필요하도록 설정해주겠습니다. 이 부분을 설정해주게 되면 wetty나 guacamole, FileZila 같은 프로그램들에서 SSH 접속이 불가능할 수 있습니다. 이 부분은 개인마다 확인 후 설정하는 것이 필요합니다.

```bash
# Google Authenticator 설치
$ sudo apt install libpam-google-authenticator
$ google-authenticator
```

Google Authenticator를 설치 후 실행해주면, Google OTP나 Authy 같은 2FA 인증 어플에서 스캔할 수 있는 QR Code를 볼 수 있습니다. 해당 QR Code를 스캔해 인증 설정을 해주면 됩니다.

QR Code와 함께 Secret Key, Secret Key 분실 시 복구를 위한 Scratch Code를 프린트합니다. 이 Scratch Code는 개인적으로 잘 보관하도록 합니다.

```bash
# ssh 설정 오픈
$ sudo vim /etc/ssh/sshd_config
```
```bash
# /etc/ssh/sshd_config에 다음과 같이 설정
# Challenge-Response Authentication 활성화
ChallengeResponseAuthentication yes
# PAM 인증 허용
UsePAM yes
# 인증 방법으로 공개키, keyboard-interactive (OTP) 설정
AuthenticationMethods publickey,keyboard-interactive
```
```bash
# pam.d 모듈의 sshd 설정
$ sudo vim /etc/pam.d/sshd
```
```bash
# /etc/pam.d/sshd 마지막 라인에 다음과 같은 설정 추가
auth required pam_google_authenticator.so nullok
```
```bash
# ssh 서비스 재구동
$ sudo systemctl restart sshd
```

설치한 Google Authenticator를 SSH 접속 시 사용하기 위해서 위와 같이 설정해줍니다. 이제 SSH 접속을 하기 위해서는 OTP 인증이 필요하게 되었습니다.

여기까지가 제가 개인 서버 구축을 진행하면서 진행한 SSH 보안 설정이었습니다. 부디 별 문제없이 서버를 잘 굴릴 수 있기를 바랍니다.

---