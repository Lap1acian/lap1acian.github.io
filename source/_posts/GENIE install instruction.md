---
layout: posts
title: Ubutnu 24.04 에 GENIE 설치하기 (24 년 09 월 기준)
date: 2025-06-04 03:55:00
categories: 
  - Research
tags:
comments: True
---

- 본 글에선 중성미자 입자 생성 시뮬레이션 프로그램 GENIE 의 설치 과정에 대해 기술한다.
- 의존성 패키지를 많이 끌고오는 프로그램이므로 Ubuntu, GENIE, 의존성 패키지 셋의 버전에 따라서 해당 방법으로 설치가 되지 않을 수도 있다.
- 사용된 ubuntu, GENIE, 의존성 패키지들의 버전은 다음과 같다.
	- ubuntu : 24.04 or 22.04
	- GENIE : R-3_04_02
	- libxml2 : 위에 기술한 ubuntu 의 apt 에서 설치
	- LHAPDF : 6.5.4
	- log4cpp : 1.1.5rc1
	- pythia : 6.4.16, 6.4.28, 8 버전을 **셋 다 사용**. 이유는 2.4. 에서 후술한다.
	- GSL : 2.8
	- ROOT : 6.30.08

2024/09/11/ docker container ubuntu 24.04 를 베이스로 한다.
2024/09/12/ docker container ubuntu 22.04 에서 동작 확인

# 1. Prerequisite apt install

apt 에서 설치해야 하는 항목

## 1.1. 필수 설치

- binutils
- cmake
- gcc
- gfortran
- git
- g++
- libgif-dev
- libgsl-dev
- libnsl-dev
- libssl-dev
- libtbb-dev
- libx11-dev
- libxext-dev
- libxft-dev
- libxml2-dev
- libxpm-dev
- locales
- python3
- rsync
- wget
- xorg

## 1.2. Integrated command

### apt 설치

```shell
apt update && \
apt install -y binutils cmake gcc gfortran git g++ libgif-dev libgsl-dev libnsl-dev libssl-dev libtbb-dev libx11-dev libxext-dev libxft-dev libxml2-dev libxpm-dev locales make python3 rsync wget xorg
```

### 로케일 설정

```shell
locale-gen en_US.UTF-8
```
# 2. Dependency 설치

## 2.1. LIBXML2

apt 에서 설치됨. Section 1.1., 1.2. 참조

## 2.2. LHAPDF - Version 6.5.4

### 2.2.1. 내려받기 및 압축 해제

```shell
cd /root
mkdir /root/lhapdf
wget https://lhapdf.hepforge.org/downloads/LHAPDF-6.5.4.tar.gz
tar xvf LHAPDF-6.5.4.tar.gz
cd /root/LHAPDF-6.5.4
```

### 2.2.2. 빌드 및 정리

- python 을 disable 하고 받아야 문제가 없다.

```shell
./configure \
	--prefix=/root/lhapdf \
	--disable-python
make -j N # N 은 장비의 스레드 개수
make install
cd /root
rm -rf LHAPDF*
```
> [!danger]
> do not use -j option on make install phase
### 2.2.3. LHAPDF integrated command

```shell
cd /root && \
mkdir /root/lhapdf && \
wget https://lhapdf.hepforge.org/downloads/LHAPDF-6.5.4.tar.gz && \
tar xvf LHAPDF-6.5.4.tar.gz && \
cd /root/LHAPDF-6.5.4 && \
./configure \
	--prefix=/root/lhapdf \
	--disable-python && \
make -j 100 && \ # N = 100 인 경우
make install && \
cd /root && \
rm -rf LHAPDF*
```
> [!danger]
> do not use -j option on make install phase
## 2.3. log4cpp - Version 1.1.5rc1

### 2.3.1. 내려받기 및 압축 해제

```shell
mkdir /root/log4cpp
mkdir /root/log4cpp_src
cd /root
wget 'https://sourceforge.net/projects/log4cpp/files/log4cpp-1.1.x (new)/log4cpp-1.1/log4cpp-1.1.5rc1.tar.gz'
tar -xvf log4cpp-1.1.5rc1.tar.gz -C /root/log4cpp_src --strip-components=2
```

### 2.3.2. 빌드 및 정리

```shell
cd log4cpp_src
./configure --prefix=/root/log4cpp
make -j N # N 은 장비의 스레드 개수
make install
cd /root
rm -rf log4cpp_* log4cpp-*
```
> [!danger]
> do not use -j option on make install phase
### 2.3.3. log4cpp integrated command

```shell
mkdir /root/log4cpp && \
mkdir /root/log4cpp_src && \
cd /root && \
wget 'https://sourceforge.net/projects/log4cpp/files/log4cpp-1.1.x (new)/log4cpp-1.1/log4cpp-1.1.5rc1.tar.gz' && \
tar -xvf log4cpp-1.1.5rc1.tar.gz -C /root/log4cpp_src --strip-components=2 && \
cd log4cpp_src && \
./configure --prefix=/root/log4cpp && \
make -j 100 && \
make install && \
cd /root && \
rm -rf log4cpp_* log4cpp-*
```
> [!danger]
> do not use -j option on make install phase

## 2.4. PYTHIA6 - Version 6.4.28

### 2.4.1. 내려받기 및 압축해제 및 전처리

Pythia6 의 경우, 빌드 시 dependency 가 되는 것을 담아놓은 pythia6 폴더와, 각 minor version  에 따른 실제 골자가 되는 .f 파일이 따로 배포된다.
본 글에서 사용하는 pythia6 폴더에는 기본적으로 6.4.16 의 .f 파일이 들어가 있는데, 이를 pythia6 의 최종 패치 버전인 6.4.28 의 .f 파일을 사용할 것이다. Root 만 설치할 경우, Pythia6 은 필요하지 않고 심지어는 현행 Root 버전에선 removal 되어 사용하지 않는 것이 옳다.
하지만 GENIE 의 최신 버전에서는 Pythia6 의 라이브러리 경로와 Pythia6 을 베이스로 빌드한 Root 에서 생성된 파일을 dependency 로 사용하고 있기 때문에 GENIE 의 설치에는 Pythia6 의 올바른 설치가 상당히 중요하다. 

```shell
# pythia6 다운로드 및 압축 해제
wget https://root.cern.ch/download/pythia6.tar.gz
tar zxvf pythia6.tar.gz
# pythia 6.4.28 의 fortran 코드 다운로드 및 6.4.16 의 fortran 코드 대체
wget https://www.pythia.org/download/pythia6/pythia6428.f
mv pythia6428.f pythia6/pythia6428.f
rm pythia6/pythia6416.f
cd /root/pythia6
```

이 상태에서 빌드하면 몇 개의 변수가 빌드 과정에서 **다른 값으로 중복 정의**되는 문제가 있다.
이는 fortran 으로 작성된 코드를 c 로 변환하는 과정에서 생기는 문제로 보인다.
실제로 Pythia6 의 매뉴얼에서는 이를 피하기 위해 external 처리해야하는 변수의 종류를 소개하고 있다.

pythia6_common_address.c 파일을 열어
52-60, 62, 64 - 65, 69, 72 번째 줄의 맨 앞에 extern 이라는 조건을 추가해야 한다.
다음의 명령어를 사용하면 된다.

```shell
sed -i '52,60s/^/extern /;62s/^/extern /;64,65s/^/extern /;69s/^/extern /;72s/^/extern /' /root/pythia6/pythia6_common_address.c
```

검토해보고 싶다면 파일을 들어갔을 때 다음과 같이 수정되어있으면 된다.

![](/attachment/img/pythia6_mod.png)

### 2.4.2. 빌드 및 정리

```shell
./makePythia6.linuxx8664 -j N  # N 은 장비의 스레드 개수
# 정리 단계
cd /root
rm pythia6.tar.gz
```

### 2.4.3. PYTHIA6 integrated command

```shell
wget https://root.cern.ch/download/pythia6.tar.gz && \
tar zxvf pythia6.tar.gz && \
wget https://www.pythia.org/download/pythia6/pythia6428.f && \
mv pythia6428.f pythia6/pythia6428.f && \
rm pythia6/pythia6416.f && \
cd /root/pythia6 && \
sed -i '52,60s/^/extern /;62s/^/extern /;64,65s/^/extern /;69s/^/extern /;72s/^/extern /' /root/pythia6/pythia6_common_address.c && \
./makePythia6.linuxx8664 -j 100 && \
cd /root && \
rm pythia6.tar.gz
```

## 2.5. PYTHIA8 - Version 8.3.12 (option for root)

Pythia8 은 root 에만 들어가는 옵션이고, GENIE 에서도 Pythia8 을 옵션으로 달 수 있지만 현재는 에러를 해결하지 못해서 GENIE 에서 enable 옵션은 사용하지 않았다.

### 2.5.1. 내려받기 및 압축해제

```shell
wget https://www.pythia.org/download/pythia83/pythia8312.tgz
mkdir /root/pythia8
tar xvf pythia8312.tgz
cd /root/pythia8312
```

### 2.5.2. 빌드 및 정리

```shell
./configure --prefix=/root/pythia8
make -j N # N 은 장비의 스레드 개수
make install
cd /root
rm -rf pythia8312*
```
> [!danger]
> do not use -j option on make install phase

### 2.5.3. PYTHIA8 integrated command

```shell
wget https://www.pythia.org/download/pythia83/pythia8312.tgz && \
mkdir /root/pythia8 && \
tar xvf pythia8312.tgz && \
cd /root/pythia8312 && \
./configure --prefix=/root/pythia8 && \
make -j 100 && \
make install && \
cd /root && \
rm -rf pythia8312*
```
> [!danger]
> do not use -j option on make install phase

## 2.6. GSL - Version 2.8

### 2.6.1. 내려받기 및 압축 해제

```shell
mkdir /root/gsl
wget https://mirror.ibcp.fr/pub/gnu/gsl/gsl-latest.tar.gz
tar xvf gsl-latest.tar.gz
cd /root/gsl-2.8
```

### 2.6.2. 빌드 및 정리

```shell
./configure --prefix=/root/gsl
make -j N # N 은 장비의 스레드 수
make install
cd /root
rm -rf gsl-2.8 gsl-latest.tar.gz
```
> [!danger]
> do not use -j option on make install phase

### 2.6.3. GSL integrated command

```shell
mkdir /root/gsl && \
wget https://mirror.ibcp.fr/pub/gnu/gsl/gsl-latest.tar.gz && \
tar xvf gsl-latest.tar.gz && \
cd /root/gsl-2.8 && \
./configure --prefix=/root/gsl && \
make -j 100 && \
make install && \
cd /root && \
rm -rf gsl-2.8 gsl-latest.tar.gz
```
> [!danger]
> do not use -j option on make install phase

## 2.7 ROOT - Version 6.30.08

GENIE 는 pythia6 의존성이 필요한데, 특히 root 에서 EGPythia6 를 불러오는데 이를 ROOT 최신버전에선 지원하지 아니함.
ROOT 에서 PYTHIA6 가 deprecate 되는 시점이 6.30, 완전히 removal 되는 시점이 6.32 이다.
따라서 2024/09/11 기준 PYTHIA6 를 지원하며 가장 최근까지 패치가 진행되었던 ROOT 6.30.08 버전을 사용한다.

### 2.7.1. 내려받기, 압축 해제 및 디렉토리 준비

```shell
cd /root
mkdir root_build && mkdir root_install # root 는 반드시 빌드 위치가 src 파일 위치와는 달라야 함.
git clone https://github.com/root-project/root.git root_src
cd root_src
git checkout v6-30-08
cd /root/root_build
```

### 2.7.2. 빌드 및 정리

```shell
cmake \
	-DCMAKE_INSTALL_PREFIX=/root/root_install \ # 설치 위치
	-DPYTHIA6_LIBRARY=/root/pythia6/libPythia6.so \ # pythia6 라이브러리 위치 (필수사항)
	-DPYTHIA8_INCLUDE_DIR=/root/pythia8/include \ # pythia8 헤더파일 위치 (선택사항1)
	-DPYTHIA8_LIBRARY=/root/pythia8/lib/libpythia8.so \ # pythia8 라이브러리 위치 (선택사항1)
	-DGSL_DIR=/usr/local \ # GSL 위치
	-Dbuiltin_xrootd=ON \ # 빌트인 x11 기반 root
	-Droofit=ON \ # roofit
	-Dpythia6=ON \
	-Dpythia8=ON \
	-Dmathmore=ON \
	-Dpython=OFF \ # pythia8 ON (선택사항1), pythia6 ON (필수사항), mathmore ON (필수사항:GENIE 가 요구함)
	../root_src
cmake --build . --target install -- -j N # N 은 장비의 스레드 수
cd /root/root_install/bin
source thisroot.sh
cd /root
```

### 2.7.3. ROOT Integrated command

```shell
cd /root && \
mkdir root_build && \
mkdir root_install && \
git clone https://github.com/root-project/root.git root_src && \
cd /root/root_src && \
git checkout v6-30-08 && \
cd /root/root_build && \
cmake \
	-DCMAKE_INSTALL_PREFIX=/root/root_install \
	-DPYTHIA6_LIBRARY=/root/pythia6/libPythia6.so \
	-DPYTHIA8_INCLUDE_DIR=/root/pythia8/include \
	-DPYTHIA8_LIBRARY=/root/pythia8/lib/libpythia8.so \
	-DGSL_DIR=/usr/local \
	-Dbuiltin_xrootd=ON \
	-Droofit=ON \
	-Dpythia6=ON \
	-Dpythia8=ON \
	-Dmathmore=ON \
	-Dpython=OFF \
	../root_src && \
cmake --build . --target install -- -j 100 && \
cd /root/root_install/bin && \
source thisroot.sh && \
cd /root
```

# 3. GENIE - Version 3.04.02

1 과 2 에서 GENIE 설치에 요구되는 요구사항을 전부 설치하였으며
이 단계에서는 본인의 연구에 필요한 최소한의 요소로 변수를 제거하고 설치하였음을 미리 밝히는 바이다.
앞서 설치한 요구사항을 GENIE 에서 제대로 인식하기 위해서 환경변수를 설정해 주어야 하는데
상기 단계를 그대로 따라왔다면 3.1. 의 echo 로 시작하는 명령줄이 이를 자동으로 진행해 줄 것이다.

## 3.1. 내려받기, 압축 해제 환경 변수 설정

```shell
mkdir /root/GENIE
git clone https://github.com/GENIE-MC/Generator.git GENIE_Generator
cd /root/GENIE_Generator
git checkout R-3_04_02
echo -e 'export GENIE=/root/GENIE_Generator\nexport ROOTSYS=/root/root_install\nexport LOG4CPP=/root/log4cpp\nexport LHAPATH=/root/lhapdf\nexport PYTHIA6=/root/pythia6\nexport PATH=/lib:$PATH:$ROOTSYS/bin:$GENIE/bin\nexport LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$LOG4CPP/lib:/lib:/usr/lib/x86_64-linux-gnu:/lib/x86_64-linux-gnu:$LHAPATH/lib:$PYTHIA6:$ROOTSYS/lib:$GENIE/lib' >> /root/.zshrc
source /root/.zshrc
```

## 3.2. 빌드 및 정리

```shell
./configure \
	--prefix=/root/GENIE \ # GENIE 의 설치 경로
	--with-lhapdf6-inc=/root/lhapdf/include \ # lhapdf 헤더 경로
	--with-lhapdf6-lib=/root/lhapdf/lib \ # lhapdf 라이브러리 경로
	--with-libxml2-inc=/usr/include/libxml2 \ # libxml2  헤더 경로 : apt 외의 수단으로 설치할 경우 달라질 수 있음
	--with-libxml2-lib=/lib/x86_64-linux-gnu \ # libxml2 라이브러리 경로 : apt 외의 수단으로 설치할 경우 달라질 수 있음
	--with-log4cpp-inc=/root/log4cpp/include \ # log4cpp 헤더 경로
	--with-log4cpp-lib=/root/log4cpp/lib \ # log4cpp 라이브러리 경로
	--with-pythia6-lib=/root/pythia6 \ # pythia6 라이브러리 경로
	--with-gfortran-lib=/lib/x86_64-linux-gnu \ # gfortran 라이브러리 경로
	--disable-lhapdf5 \
	--enable-lhapdf6 \
	--enable-gfortran # lhapdf5 말고 lhapdf6 사용, gfortran 사용
make -j N # N 은 장비의 스레드 수
make install
```
> [!danger]
> do not use -j option on make install phase

## 3.3. GENIE integrated command

```shell
mkdir /root/GENIE && \
git clone https://github.com/GENIE-MC/Generator.git GENIE_Generator && \
cd /root/GENIE_Generator && \
git checkout R-3_04_02 && \
echo -e 'export GENIE=/root/GENIE_Generator\nexport ROOTSYS=/root/root_install\nexport LOG4CPP=/root/log4cpp\nexport LHAPATH=/root/lhapdf\nexport PYTHIA6=/root/pythia6\nexport PYTHIA8=/root/pythia8\nexport PATH=/lib:$PATH:$ROOTSYS/bin:$GENIE/bin\nexport LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$LOG4CPP/lib:/lib:/usr/lib/x86_64-linux-gnu:/lib/x86_64-linux-gnu:$LHAPATH/lib:$PYTHIA6:$PYTHIA8/lib:$ROOTSYS/lib:$GENIE/lib' >> /root/.zshrc && \
source /root/.zshrc && \
./configure \
	--prefix=/root/GENIE \
	--with-lhapdf6-inc=/root/lhapdf/include \
	--with-lhapdf6-lib=/root/lhapdf/lib \
	--with-libxml2-inc=/usr/include/libxml2 \
	--with-libxml2-lib=/lib/x86_64-linux-gnu \
	--with-log4cpp-inc=/root/log4cpp/include \
	--with-log4cpp-lib=/root/log4cpp/lib \
	--with-pythia6-lib=/root/pythia6 \
	--with-pythia8-lib=/root/pythia8/lib \
	--with-gfortran-lib=/lib/x86_64-linux-gnu \
	--disable-lhapdf5 \
	--enable-lhapdf6 \
	--disable-pythia6 \
	--enable-pythia8 \
	--enable-gfortran && \
make -j 100 && \
make install
```
> [!danger]
> do not use -j option on make install phase

# test

gevgen_hadron -n 10000 -p 211 -t 1000080160 -k 0.2 --seed 9839389

```shell
cd /root && \
mkdir /root/lhapdf && \
wget https://lhapdf.hepforge.org/downloads/LHAPDF-6.5.4.tar.gz && \
tar xvf LHAPDF-6.5.4.tar.gz && \
cd /root/LHAPDF-6.5.4 && \
./configure \
	--prefix=/root/lhapdf \
	--disable-python && \
make -j 100 && \ # N = 100 인 경우
make install && \
cd /root && \
rm -rf LHAPDF* && \
mkdir /root/log4cpp && \
mkdir /root/log4cpp_src && \
cd /root && \
wget 'https://sourceforge.net/projects/log4cpp/files/log4cpp-1.1.x (new)/log4cpp-1.1/log4cpp-1.1.5rc1.tar.gz' && \
tar -xvf log4cpp-1.1.5rc1.tar.gz -C /root/log4cpp_src --strip-components=2 && \
cd log4cpp_src && \
./configure --prefix=/root/log4cpp && \
make -j 100 && \
make install && \
cd /root && \
rm -rf log4cpp_* log4cpp-* && \
wget https://root.cern.ch/download/pythia6.tar.gz && \
tar zxvf pythia6.tar.gz && \
wget https://www.pythia.org/download/pythia6/pythia6428.f && \
mv pythia6428.f pythia6/pythia6428.f && \
rm pythia6/pythia6416.f && \
cd /root/pythia6 && \
sed -i '52,60s/^/extern /;62s/^/extern /;64,65s/^/extern /;69s/^/extern /;72s/^/extern /' /root/pythia6/pythia6_common_address.c && \
./makePythia6.linuxx8664 -j 100 && \
cd /root && \
rm pythia6.tar.gz && \
wget https://www.pythia.org/download/pythia83/pythia8312.tgz && \
mkdir /root/pythia8 && \
tar xvf pythia8312.tgz && \
cd /root/pythia8312 && \
./configure --prefix=/root/pythia8 && \
make -j 100 && \
make install && \
cd /root && \
rm -rf pythia8312* && \
mkdir /root/gsl && \
wget https://mirror.ibcp.fr/pub/gnu/gsl/gsl-latest.tar.gz && \
tar xvf gsl-latest.tar.gz && \
cd /root/gsl-2.8 && \
./configure --prefix=/root/gsl && \
make -j 100 && \
make install && \
cd /root && \
rm -rf gsl-2.8 gsl-latest.tar.gz && \
cd /root && \
mkdir root_build && \
mkdir root_install && \
git clone https://github.com/root-project/root.git root_src && \
cd /root/root_src && \
git checkout v6-30-08 && \
cd /root/root_build && \
cmake \
	-DCMAKE_INSTALL_PREFIX=/root/root_install \
	-DPYTHIA6_LIBRARY=/root/pythia6/libPythia6.so \
	-DPYTHIA8_INCLUDE_DIR=/root/pythia8/include \
	-DPYTHIA8_LIBRARY=/root/pythia8/lib/libpythia8.so \
	-DGSL_DIR=/usr/local \
	-Dbuiltin_xrootd=ON \
	-Droofit=ON \
	-Dpythia6=ON \
	-Dpythia8=ON \
	-Dmathmore=ON \
	-Dpython=OFF \
	../root_src && \
cmake --build . --target install -- -j 100 && \
cd /root/root_install/bin && \
source thisroot.sh && \
cd /root && \
mkdir /root/GENIE && \
git clone https://github.com/GENIE-MC/Generator.git GENIE_Generator && \
cd /root/GENIE_Generator && \
git checkout R-3_04_02 && \
echo -e 'export GENIE=/root/GENIE_Generator\nexport ROOTSYS=/root/root_install\nexport LOG4CPP=/root/log4cpp\nexport LHAPATH=/root/lhapdf\nexport PYTHIA6=/root/pythia6\nexport PATH=/lib:$PATH:$ROOTSYS/bin:$GENIE/bin\nexport LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$LOG4CPP/lib:/lib:/usr/lib/x86_64-linux-gnu:/lib/x86_64-linux-gnu:$LHAPATH/lib:$PYTHIA6:$ROOTSYS/lib:$GENIE/lib' >> /root/.zshrc && \
source /root/.zshrc && \
./configure \
	--prefix=/root/GENIE \
	--with-lhapdf6-inc=/root/lhapdf/include \
	--with-lhapdf6-lib=/root/lhapdf/lib \
	--with-libxml2-inc=/usr/include/libxml2 \
	--with-libxml2-lib=/lib/x86_64-linux-gnu \
	--with-log4cpp-inc=/root/log4cpp/include \
	--with-log4cpp-lib=/root/log4cpp/lib \
	--with-pythia6-lib=/root/pythia6 \
	--with-gfortran-lib=/lib/x86_64-linux-gnu \
	--disable-lhapdf5 \
	--enable-lhapdf6 \
	--enable-gfortran && \
make -j 100 && \
make install && \
echo 'INSTALL COMPELETE'

```

