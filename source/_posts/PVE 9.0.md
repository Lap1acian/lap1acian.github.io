---
layout: posts
title: PVE 9.0.11 환경에서 nvidia 드라이버 설치 및 lxc 에 gpu 공유 설정
date: 2024-07-24 10:05:52
categories:
  - Research
tags:
  - subtitle_works
comments: true
---

# PVE 9.0.11 환경에서 nvidia 삽질기

참고 문헌

Proxmox LXC에 Nvidia GPU 패스쓰루: https://svrforum.com/os/503310

예전에 이 글을 보고 따라간 적이 있었는데 몇몇 부분에서 보충이 필요한 부분이 있어 찾아가면서 했던 경험이 있다. 당시에 글로 남길까 했는데 그 때는 일단 진행해놓고 보자는 생각으로 지나왔더니 까먹고 글을 안 썼더랬다. 그 때의 업보인가 똑같은 짓을 다시 할 필요가 생겨서 이번에는 두 번 실수하지 않으리라 결심하고 글을 쓴다. 

버전 목록
PVE 9.0.11
NVIDIA 드라이버 버전 : NVIDIA-Linux-x86_64-580.105.08


# 0. 사전 설정

proxmox 를 처음 설치한 경우, enterprise repository 가 기본 레포지토리로 설정되어있다.
이 때 apt update 를 수행하면 우리는 enterprise repository 가 구독되어 있지 않으므로 일부 레포지토리를 불러오지 못한다.
따라서 이 문제를 해결하려면 repository 를 no subcription repository 로 수정해 주어야 한다.
이는 pve(proxmox virtual environment) 의 커널을 최신 버전으로 업데이트하고 nvidia driver 설치를 진행하게 되는데 이를 위해선 proxmox 쪽 repository 를 올바르게 불러와야하기 때문이다.
이 문제를 해결한 사용자는 다음 섹션으로 바로 넘어가도 좋다.

pve.proxmox.com/wiki 의 package repositories 항목에서 [no subcription repository](https://pve.proxmox.com/wiki/Package_Repositories#sysadmin_no_subscription_repo) 항목에 들어가보면 apt 에서 불러오는 repository 의 수정 방법이 적혀 있다. pve 9.0.11 버전 외에서의 설정 방법은 위 링크를 참조하도록 하자.

![fig1](/attachment/img/9_fig1.png)

나는 nano 보다 vim 이 편해서 일단 apt update 로 업데이트 한 번 하고 (proxmox enterprise repo 를 못 불러오는거지 다른 repo 는 불러올 수 있다.) vim 깔아서 다음 두 파일을 수정했다.

```shell
/etc/apt/source.list.d/proxmox.sources
/etc/apt/source.list.d/ceph.sources
```

proxmox.sources 의 경우 원래 파일 이름이 proxmox-enterprise.sources 이렇게 되어있을 수 있는데 mv 로 이름 바꾸고 수정해도 문제 없다.

9.0.11 버전 기준으로 proxmox.sources 의 파일 내용은 다음과 같을 것인데

```shell
Types: deb
URIs: https://enterprise.proxmox.com/debian/pve
Suites: trixie
Components: pve-enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

원 파일 내용을 주석처리하고 다음과 같이 저장하고 나오도록 하자.

```shell
# Types: deb
# URIs: https://enterprise.proxmox.com/debian/pve
# Suites: trixie
# Components: pve-enterprise
# Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg

Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

ceph.sources 도 마찬가지로

```shell
Types: deb
URIs: https://enterprise.proxmox.com/debian/ceph-squid
Suites: trixie
Components: enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

이런 내용일텐데

```shell
# Types: deb
# URIs: https://enterprise.proxmox.com/debian/ceph-squid
# Suites: trixie
# Components: enterprise
# Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg

Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

이렇게 바꿔주도록 하자.
이후 apt update 를 수행하면 pve 쪽 repository 도 잘 불러올 것이다.



# 1. proxmox 커널 버전 확인 및 업데이트
 
pve 의 쉘에서 다음을 통해 pve 커널의 버전을 확인한다.

```shell
uname -r
```

![fig2][/attachment/img/9_fig2.png]

내 경우는 6.14.11-4 버전이다.
최신 버전의 커널인지 확인하기 위해 다음을 수행한다.

```shell
apt-cache search pve-header
```

![fig3][/attachment/img/9_fig3.png]

최신 버전의 커널을 찾아 apt install 로 설치해준다. 내 경우는 6.17.2-1-pve 일 것이다.

```shell
apt install proxmox-headers-xxx-pve
```

# 2. pve 쉘에서 GPU 드라이버 블랙리스트 설정

설치가 완료되면  다음은 nvidia 드라이버를 설치해야 하는데 그 전에 proxmox 쉘에서 GPU 를 쓰면 곤란하니 

```shell
/etc/modprobe.d/blacklist.conf
```

파일을 vim 으로 열어서

```shell
blacklist nouveau
```

를 입력한 후 저장한 뒤 다음 명령어로 초기 램 파일 시스템을 업데이트하고 재부팅한다.

```shell
update-initramfs -u
reboot
```

# 3. signing 을 위한 key-pair 생성

이 부분은 4. 를 시도해보고

![fig8][/attachment/img/9_fig8.png]

이러한 문구가 뜨는 경우 필요할 수 있다. (뜨지 않는다면 스킵해도 무방하다.)

서버 환경에서 UEFI secure boot 가 설정되어 있다면 드라이버 설치 중 커널 모듈에 signing 을 강제하는 경우가 있다. secure boot 를 끄면 install without signing 을 선택할 수 있으리라 생각된다. 내 경우는 그럴 상황은 못 되니 key-pair 를 생성해서 signing 을 진행할 것이다.
# 4. Nvidia 드라이버 설치

드라이버 빌드를 위한 기본 build-essential 을 apt 로 설치한다.

```shell
apt install build-essential
```

[nvidia 드라이버 다운로드 사이트](https://www.nvidia.com/Download/index.aspx) 에서 본인의 gpu 모델에 맞는 드라이버를 찾는다.

![fig4][/attachment/img/9_fig4.png]

Find 를 누르면 최신 버전의 드라이버가 뜨는데 거기서 view  를 누르면

![fig5][/attachment/img/9_fig5.png]

이렇게 다운로드 링크가 나오는데 저걸 바로 받는 것이 아니고 오른쪽 마우스 버튼을 눌러 링크를 복사한 뒤 쉘로 돌아가 wget 으로 pve 에 받는다.

```shell
wget 복사한_링크
```

받은 파일에 실행 권한을 주기 위해 다음 명령어를 입력한다.

```shell
chmod +x 받은_파일_이름
```

이후 설치를 위해 ./받은_파일_이름 명령을 입력해주면 되는데 내 환경에서는 kernel_path 를 지정해주지 않으면 드라이버 설치 파일이 커널을 못잡는 오류가 있었다. 따라서 --kernel-source-path 옵션을 주어야 한다.

```shell
./받은_파일_이름 --kernel-source-path=/usr/src/linux-headers-x.xx.x-x.pve
```

내 경우엔 앞에 apt 로 설치한 kernel 이 저 위치에 있었다.

![fig5][/attachment/img/9_fig6.png]

MIT/GPL 선택 후 

![fig5][/attachment/img/9_fig7.png]

Kernel module 을 빌드한다

![fig5][/attachment/img/9_fig9.png]

X 를 쓰겠냐는 말인데 여기선 no 를 선택한다.

설치가 완료되었다는 박스가 뜨면 nvidia-smi 명령어를 통해 gpu 가 제대로 설치되었는지 확인할 수 있다.

```shell
nvidia-smi
```

![fig5][/attachment/img/9_fig10.png]
이렇게 설치된 gpu 의 정보가 나오면 성공이다.

이제 proxmox  가 시작될 때 nvidia 관련 드라이버가 실행되기 위해 필요한 작업으로, nvidia 관련 드라이버들을 모듈에 추가한다. 다음 파일을 vim 으로 연다

```shell
/etc/modules-load.d/modules.conf
```

파일에 다음 줄들을 추가한다

```shell
nvidia  
nvidia-modeset  
nvidia_uvm
```

![fig5][/attachment/img/9_fig11.png]

이런 꼴이 될 것이다.
initramfs 를 다시 한 번 업데이트한다.

```shell
update-initramfs -u
```
