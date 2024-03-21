FROM ubuntu:22.04

ENV DEBIAN_FRONTEND="noninteractive" \
    NVIDIA_VISIBLE_DEVICES=all \
    NVIDIA_DRIVER_CAPABILITIES=compute,utility,graphics,video

RUN echo "Europe/Brussels" > /etc/localtime && \
    echo "Europe/Brussels" > /etc/timezone

# Development APT packages
RUN apt -qq update && \
    apt -qq install -y --no-install-recommends \
        sudo \
        nano \
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
        ssh \
        python3 \
        software-properties-common \
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

# Install Vulkan dependencies
RUN apt update && \
    export DEBIAN_FRONTEND=noninteractive && \
    apt -y install \
        pciutils \
        vulkan-tools \
        mesa-utils

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

# Install ROS
ARG ROS_DISTRO=humble
RUN \
    # Setup locale
    sudo apt update && sudo apt install locales && \
    sudo locale-gen en_US en_US.UTF-8 && \
    sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
    export LANG=en_US.UTF-8 && \
    # Add the ROS 2 apt repository
    sudo add-apt-repository universe && \
    sudo apt update && sudo apt install curl -y && \
    sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    sudo apt update && \
    sudo apt install -y --no-install-recommends \
        ros-dev-tools \
        ros-humble-desktop \
        ros-humble-ros-base 

# Install ROS packages
RUN \
    sudo apt update && \
    sudo apt install -y --no-install-recommends \
        ros-${ROS_DISTRO}-slam-toolbox \
        ros-${ROS_DISTRO}-navigation2 \
        ros-${ROS_DISTRO}-nav2-bringup \
        ros-${ROS_DISTRO}-pointcloud-to-laserscan \
        ros-${ROS_DISTRO}-gazebo-msgs \
        ros-${ROS_DISTRO}-ackermann-msgs \
        ros-${ROS_DISTRO}-rmw-cyclonedds-cpp \
        ros-${ROS_DISTRO}-control-toolbox \
        ros-${ROS_DISTRO}-nav-msgs \
        ros-${ROS_DISTRO}-desktop


# Install O3DE
ENV DATA_FOLDER=/data
ENV PROJECT_PATH=$DATA_FOLDER/Ros2Project

ARG O3DE_REPO=https://github.com/o3de/o3de.git
ARG O3DE_BRANCH=2310.2


ARG O3DE_EXTRAS_REPO=https://github.com/o3de/o3de-extras.git
ARG O3DE_EXTRAS_BRANCH=2310.2

ARG IMAGE_TYPE=simulation  # Default to 'simulation'

# Clone and register O3DE repos 
RUN \
    # Clone and install O3DE
    cd $DATA_FOLDER && \
    git clone --recursive $O3DE_REPO && \
    git -C $DATA_FOLDER/o3de checkout $O3DE_BRANCH &&\
    git -C $DATA_FOLDER/o3de lfs install && \
    git -C $DATA_FOLDER/o3de lfs pull && \
    $DATA_FOLDER/o3de/python/get_python.sh && \
    $DATA_FOLDER/o3de/scripts/o3de.sh register -ep $DATA_FOLDER/o3de && \
    # Clone and install O3DE extras
    git clone $O3DE_EXTRAS_REPO && \
    git -C $DATA_FOLDER/o3de-extras checkout $O3DE_EXTRAS_BRANCH && \
    $DATA_FOLDER/o3de/scripts/o3de.sh register -gp $DATA_FOLDER/o3de-extras/Gems/ROS2 && \
    $DATA_FOLDER/o3de/scripts/o3de.sh register -gp $DATA_FOLDER/o3de-extras/Gems/RosRobotSample && \
    $DATA_FOLDER/o3de/scripts/o3de.sh register -gp $DATA_FOLDER/o3de-extras/Gems/WarehouseAssets && \
    $DATA_FOLDER/o3de/scripts/o3de.sh register -gp $DATA_FOLDER/o3de-extras/Gems/WarehouseSample

# Build and cleanup in the same layer to reduce the size
RUN if [ "${IMAGE_TYPE}" = "simulation" ]; then \
        . /opt/ros/${ROS_DISTRO}/setup.sh && \
        $DATA_FOLDER/o3de/scripts/o3de.sh create-project --project-path $PROJECT_PATH --template-path $DATA_FOLDER/o3de-extras/Templates/Ros2ProjectTemplate && \
        cmake -B $PROJECT_PATH/build/linux -S $PROJECT_PATH -G "Ninja Multi-Config" -DLY_STRIP_DEBUG_SYMBOLS=TRUE -DLY_DISABLE_TEST_MODULES=ON && \
        cmake --build $PROJECT_PATH/build/linux --config profile --target Ros2Project Editor Ros2Project.Assets Ros2Project.GameLauncher && \
        $DATA_FOLDER/cleanup.bash ; \llation
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