FROM ubuntu:22.04

# Development APT packages
RUN apt -qq update && \
    apt -qq install -y --no-install-recommends \
        sudo \
        git \
        wget \
        curl \
        zip \
        unzip \
        tar \
        ca-certificates \
        build-essential \
        gcc \
        g++ \
        gdb \
        make \
        ninja-build \
        autoconf \
        automake \
        locales-all \
        dos2unix \
        rsync \
        tar \
        ssh \
        python3 \
        && \
    apt -qq clean && \
    apt -qq autoclean && \
    apt -qq remove && \
    apt -qq autoremove

# Install CMAKE
ENV CMAKE_VERSION 3.28.3
RUN apt update && \
    apt -qq install -y --no-install-recommends \
        graphviz \
    && \
    cd /tmp && \
    wget --no-check-certificate https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz && \
    tar -xzvf cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz && \
    mv cmake-${CMAKE_VERSION}-linux-x86_64 /opt/cmake && \
    rm cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz
ENV PATH="/opt/cmake/bin:${PATH}"

# Download and install Clang binaries
RUN apt update && \
    apt -qq install -y \ 
    libstdc++-12-dev \
    clang

# Install O3DE dependencies
RUN apt update && \
    apt -qq install -y \
        libglu1-mesa-dev \
        libxcb-xinerama0 \
        libxcb-xinput0 \
        libxcb-xinput-dev \
        libxcb-xfixes0-dev \
        libxcb-xkb-dev \
        libxkbcommon-dev \
        libxkbcommon-x11-dev \
        libfontconfig1-dev \
        libpcre2-16-0 \
        zlib1g-dev \
        mesa-common-dev \
        libunwind-dev \
        libzstd-dev \
        && \
    apt -qq clean && \
    apt -qq autoclean && \
    apt -qq remove && \
    apt -qq autoremove

# Install XServer dependencies
RUN apt-get update && apt-get install -y --install-recommends \
    # x general
    libxrandr-dev \
    libxinerama-dev \
    libxcursor-dev \
    xauth \
    libx11-dev \
    libx11-xcb-dev \
    libxi-dev \
    libxcb1-dev \
    libxcb-randr0-dev \
    libxfixes-dev \
    libxss-dev \
    libxkbcommon-dev \
    libxcb-icccm4 libxcb-image0 libxcb-keysyms1 libxcb-randr0 libxcb-render-util0 libxcb-xinerama0 libxcb-xkb1 libxkbcommon-x11-0 libxcb-xinput0

# Create User
RUN \
    useradd -u 1000 dev && \
    echo "dev:dev" | chpasswd && \
    adduser dev sudo && \
    mkdir -p /home/dev && \
    mkdir -p /home/dev/Documents && \
    mkdir -p /home/dev/Downloads && \
    chown -R dev /home/dev && \
    printf "\ndev ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
ENV \
    USER=dev \
    PATH=/home/dev/.local/bin:$PATH

USER dev
WORKDIR /home/dev

# Install ZSH
RUN sudo apt install -y zsh && \
    sudo chsh -s /usr/bin/zsh && \
    # Install oh my zsh
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" && \
    # configure .zshrc
    cd ~ && \
    touch .zshrc && \
    sed -i 's/ZSH_THEME=.*/ZSH_THEME="eastwood"/' .zshrc && \
    echo "ZSH Installed"

ENTRYPOINT ["/usr/bin/zsh"]

WORKDIR /home/benjamin