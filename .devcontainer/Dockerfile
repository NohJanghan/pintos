FROM ubuntu:14.04

# 필수 패키지 및 매뉴얼 요구 환경 설치
RUN apt-get update && apt-get install -y \
    gcc-4.4 \
    g++ \
    qemu \
    wget \
    tar \
    make \
    vim \
    build-essential \
    xorg-dev \
    libncurses5-dev \
    git \
    && rm -rf /var/lib/apt/lists/*

# gcc-4.4를 기본 gcc로 심볼릭 링크 설정
RUN mv /usr/bin/gcc-4.4 /usr/bin/gcc || true

# bochs-2.6.2 다운로드 및 빌드
WORKDIR /tmp
RUN wget -O bochs-2.6.2.tar.gz https://sourceforge.net/projects/bochs/files/bochs/2.6.2/bochs-2.6.2.tar.gz && \
    tar xvf bochs-2.6.2.tar.gz && \
    cd bochs-2.6.2 && \
    ./configure --enable-gdb-stub --with-nogui && \
    make && \
    make install && \
    rm -rf /tmp/bochs-2.6.2*

# VS Code DevContainer가 마운트할 기본 작업 공간 설정
RUN mkdir -p /workspaces/pintos-project
WORKDIR /workspaces/pintos-project

# PATH 설정
ENV PATH="${PATH}:/workspaces/pintos-project/pintos/src/utils"