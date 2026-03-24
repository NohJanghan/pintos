FROM ubuntu:14.04

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

# make symlink
RUN mv /usr/bin/gcc-4.4 /usr/bin/gcc || true

# bochs-2.6.2 download and build
WORKDIR /tmp
RUN wget -O bochs-2.6.2.tar.gz https://sourceforge.net/projects/bochs/files/bochs/2.6.2/bochs-2.6.2.tar.gz && \
    tar xvf bochs-2.6.2.tar.gz && \
    cd bochs-2.6.2 && \
    ./configure --enable-gdb-stub --with-nogui && \
    make && \
    make install && \
    rm -rf /tmp/bochs-2.6.2*

# VS Code DevContainer mount default workspace
RUN mkdir -p /workspaces/pintos-project
WORKDIR /workspaces/pintos-project

# set PATH
ENV PATH="${PATH}:/workspaces/pintos-project/pintos/src/utils"