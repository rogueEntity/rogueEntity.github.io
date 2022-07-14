---
title:  "Mac OS Setup"
excerpt: "개인용 맥북 셋업 가이드."

date: 2022-07-14
last_modified_at: 2022-07-14

categories:
  - Setup
tags:
  - Setup
  - Mac
  - OS

toc: true
toc_sticky: true
---

# Mac OS Setting

## Intro
→ 최근에 맥북을 샀음. 주변에서 구한 2019년형 Intel Macbook Pro인데 Mac OS 써보는게 처음임. Windows나 linux 사용할 때처럼 Mac에서도 기본 세팅부터 하고 쓸건데, 나중에 OS 재설치하게 되면 그때 다시 참고하려고 정리함.

---

## Settings
### System
1. Synchro
    ```
    시스템 환경설정
    → Apple ID
    → iCloud Drive
    → 데스크탑 및 문서 폴더 동기화 해제
    ```
2. Dock
    ```
    시스템 환경설정
    → Dock
    → 자동으로 Dock 가리기
    ```
3. Sound Control
    ```
    시스템 환경설정
    → 사운드
    → 메뉴 막대에서 음량 보기
    ```
4. Battery
    ```
    시스템 환경설정
    → 에너지 절약
    → 메뉴 막대에서 배터리 상태 보기
    ```
5. Finder
    ```
    시스템 환경설정
    → 일반
    → 스크롤 막대 보기
    → 항상
    
    Finder
    → 보기
    → 상태 막대 보기
    
    Finder
    → 환경설정
    → 고급
    → 폴더 우선 정렬
    → 윈도우에서(이름순으로 정렬시)
    ```
---

## Installation
### Homebrew
```sh
# homebrew
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
$ brew update
```
```sh
# cask
$ brew install cask
```
→ Homebrew로 다들 프로그램 설치를 한다길래 나도 써봐야겠으니까 Homebrew 설치를 해주고, GUI 프로그램을 위한 cask까지 설치함.

### iterm2
```sh
# zsh
$ brew upgrade zsh
$ zsh --version
```
```sh
# iterm2
$ brew install --cask iterm2
```
→ 기본 터미널보단, iterm2라는 터미널을 사용하는게 국룰인 것 같음. 기본으로 설치되어 있는 zsh를 upgrade하고, iterm2도 설치함.

### Oh My Zsh
```sh
# Oh My Zsh
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
```sh
# Powerlevel10k
$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
```
```sh
$ p10k configure
```
→ 터미널만 켜도 바로 프로그래밍이 하고 싶어지는 테마를 쓰려면, Oh My Zsh를 설치하고 powerlevel10k라는 테마도 받아줌. powerlevel10k를 내려받았으면, iterm2를 재시작하고 테마 설정을 잡아줘야함.

### Font
→ D2Coding 폰트도 설치하고 설정할 거임. iterm2 설정에서 폰트를 바꿔줄건데, iterm2 폰트 설정에서 D2Coding을 선택해주고 `Use a different font for non-ASCII text` 옵션을 체크해줘야 함. non-ASCII 폰트는 `MesloLGS NF` 폰트로 설정함.

### Syntax Highlighting
```sh
# zsh-syntax-highlighting
$ brew install zsh-syntax-highlighting
$ vim ~/.zshrc
```
```sh
source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```
→ syntax hilighting을 설치해주고 intel Mac이기 때문에, 위 한 줄을 zshrc 마지막에 추가해 알아서 실행되도록 함.

### Neofetch
```sh
# neofetch
$ brew install neofetch
$ vim ~/.zshrc
```
```sh
neofetch
```
```sh
$ vim ~/.p10k.zsh
```
```sh
# modify this value (verbose → off)
# POWERLEVEL9K_INSTANT_PROMPT=off
# ---
# neofetch with image
$ cd /Users/(whoami)/.config/neofetch
$ vim config.conf
# search properties with '/image'
# image_backend="iterm2"
# image_source="(image path)"
# image_size="(image size)px"
# ---
```
→ 이제 막바지로 터미널 뽕이 차오를 수 있게 neofetch도 설치함. 터미널이 켜지면 neofetch도 바로 실행될 수 있도록 zshrc 마지막 줄에 `neofetch`라고 한 줄을 추가함.

powerlevel 테마에서 instant prompt라는 옵션이 있는데, 이걸 verbose에서 off로 바꿔줌. Powerlevel10k는 속도 때문에 비동기로 처리되는 부분이 많은데, 동기 처리되는 작업 때문에 생기는 시동 지연시간을 없애고자 존재하는 옵션임. ASCII 로고 대신 이미지를 쓴다던가 하면 이것 때문에 똑바로 동작하지 않는 경우가 생김.

### Tree
```sh
# tree
$ brew install tree
```
→ tree는 뭐... 쓸 일이 있을 것 같아서 설치했음.

### Utility
```sh
# scroll-reverser
$ brew install --cask scroll-reverser

# alt-tab
$ brew install --cask alt-tab

# hiddenbar
$ brew install --cask hiddenbar

# rectangle
$ brew install --cask rectangle

# appcleaner
$ brew install --cask

#cheatsheet
$ brew install --cask cheatsheet

# Runcat
# System Resource Monitoring
# install on Mac App Store

# iright-mouse
# Context Menu Controll
# install on Mac App Store
```
→ 사용 편의를 생각해서 몇가지 프로그램들을 설치함. brew를 적극적으로 써보려고 최대한 brew로 설치했음.

```sh
# keka
$ brew install --cask keka

# iina
$ brew install --cask iina

# fliqlo
$ brew install --cask fliqlo

# Dynamic wallpaper
# install on Mac App Store

# Naver Whale
$ brew install --cask naver-whale
```
→ 브라우저, 압축 프로그램, 영상 플레이어 같이 뭔가 있어야 할 것 같은 프로그램도 설치함. 나중에 다른 프로그램으로 변경할지도 모르겠음. 일단 추천 받은 프로그램으로 설치함.

### Programs

- <span style="color:brown">Note</span>
    - Notion
- <span style="color:brown">Dev</span>
    - VSCode
- <span style="color:brown">Messenger</span>
    - Discord
    - 카카오톡

→ 여기서부터는 혹시나 나중에 OS 재설치 같은 일이 있을 때 참고할 용도로 기록함. 이 리스트가 제일 많이 변할 것 같음.

---

## Vim
### NeoVim

```sh
# not yet
```
→ NeoVim에 플러그인을 잔뜩 설치해놓고 IDE로 사용하지...는 않을 것 같음. 근데 일단 만지작거리고 싶은 마음이 있긴 해서, 나중에 시도해보고 여기에 정리할 생각임.

---
