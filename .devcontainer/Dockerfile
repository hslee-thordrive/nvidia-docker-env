FROM thorenv/nvidia:ros-kinetic

#===>> installs git
RUN apt-get -qq update \
 && apt-get install -y software-properties-common \
 && add-apt-repository -y ppa:git-core/ppa \
 && apt-get upgrade -y \
 && apt-get update \
 && apt-get install -y git

#===>> installs git-lfs
RUN apt-get -qq update \
 && apt-get -qq install -y curl \
 && curl -s "https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh" | bash \
 && apt-get -qq install -y git-lfs

#===>> installs cmake
ARG CMAKE_VERSION="3.21.1"
RUN apt-get -qq update
RUN apt-get -qq install -y curl tar
RUN curl -L -o /tmp/cmake.tar.gz "https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz"
RUN mkdir -p /tmp/cmake
RUN tar -xzf /tmp/cmake.tar.gz \
  	   --directory=/tmp/cmake \
  	   --strip-components 1 \
  	   --exclude bin/cmake-gui \
  	   --exclude doc/cmake \
  	   --exclude "share/cmake-*/Help"
RUN cp -rf /tmp/cmake/* /usr
RUN rm -rf /tmp/cmake
RUN rm /tmp/cmake.tar.gz

#===>> installs sqlite-modern-cpp
ARG SQLITE_MODERN_CPP_VERSION="3.2"
RUN apt-get -qq update \
 && apt-get -qq install -y sqlite3 libsqlite3-dev \
 && git clone "https://github.com/SqliteModernCpp/sqlite_modern_cpp.git" /tmp/sqlite_modern_cpp \
      --depth 1 \
      --branch v${SQLITE_MODERN_CPP_VERSION} \
      --quiet \
      --progress \
 && cd /tmp/sqlite_modern_cpp \
 && ./configure \
 && make -j $(nproc) \
 && make install \
 && rm -rf /tmp/sqlite_modern_cpp

#===>> installs geographiclib
ARG GEOGRAPHICLIB_VERSION="1.49"
RUN apt-get -qq update \
 && apt-get -qq install -y curl tar wget \
 && curl -L -o /tmp/geographiclib.tar.gz "https://sourceforge.net/projects/geographiclib/files/distrib/GeographicLib-${GEOGRAPHICLIB_VERSION}.tar.gz" \
 && tar -xzf /tmp/geographiclib.tar.gz \
      --directory=/tmp \
      --keep-old-files \
 && mkdir -p /tmp/GeographicLib-1.49/build \
 && cd /tmp/GeographicLib-1.49/build \
 && cmake .. \
 && make -j $(nproc) \
 && make install \
 && cd /tmp/GeographicLib-1.49 \
 && ./configure \
 && ldconfig \
 && geographiclib-get-geoids best \
 && rm -rf /tmp/geographiclib \
 && rm /tmp/geographiclib.tar.gz

#===>> upgrades gcc
RUN apt-get -qq update \
 && apt-get -qq install -y build-essential software-properties-common \
 && add-apt-repository ppa:ubuntu-toolchain-r/test -y \
 && apt-get -qq update \
 && apt-get -qq install -y gcc-9 g++-9\
 && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 60 --slave /usr/bin/g++ g++ /usr/bin/g++-9 \
 && update-alternatives --config gcc

RUN apt-get -qq update \
 && apt-get -qq install -y gedit \
                           gcovr \
                           doxygen \
                           graphviz \
                           clang-tidy \
                           mesa-utils \
                           bash-completion \
                           python3-pip \
                           python3-dev \
                           libssl-dev \
                           libpcap-dev \
                           libgl1-mesa-glx \
                           libyaml-cpp-dev \
                           libeigen3-dev \
                           libpcl-dev \
                           libtbb-dev

#===>> cleaning
RUN apt-get -qq update \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /var/tmp/* \
 && rm -rf /tmp/*

#===>> format setup
RUN curl -sL -o "/.clang-format" "https://raw.githubusercontent.com/thordrive/conventions/master/cpp/.clang-format"

#===>> user setup
ARG USERNAME=thordrive
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN groupadd --gid ${USER_GID} ${USERNAME} \
 && useradd --uid ${USER_UID} --gid ${USER_GID} -m ${USERNAME} \
 && apt-get -qq update \
 && apt-get -qq install -y sudo \
 && echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME} \
 && echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL >> /etc/sudoers \
 && chmod 0440 /etc/sudoers.d/${USERNAME}

USER ${USERNAME}

#===>> ros setup
ARG ROS_DISTRIBUTION="kinetic"
RUN sudo apt-get -qq update \
 && sudo rm -rf /etc/ros/rosdep/sources.list.d/20-default.list \
 && sudo rosdep init \
 && rosdep update
ADD default.rviz /home/${USERNAME}/.rviz/default.rviz

#===>> git setup
RUN git config --global pager.branch false \
 && git config --global color.branch.current "#FF0000" \
 && git config --global color.branch.local "#FFFF00" \
 && git config --global color.branch.plain "#000000" \
 && git config --global color.branch.remote "#00FF00" \
 && git config --global color.branch.upstream "#0000FF"

#===>> font setup
RUN mkdir -p /home/${USERNAME}/.local/share/fonts
ADD fonts/FiraCode-*.ttf /home/${USERNAME}/.local/share/fonts
RUN fc-cache -f

#===>> language setup
RUN sudo apt-get -qq update \
 && sudo apt-get -qq install language-pack-en \
 && sudo locale-gen en_US \
 && sudo update-locale

#===>> zsh setup
RUN sudo apt-get -qq update \
 && sudo apt-get -qq install -y zsh \
 && wget "https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh" -O - | zsh || true \
 && git clone "https://github.com/zsh-users/zsh-autosuggestions" ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
 && git clone "https://github.com/zsh-users/zsh-syntax-highlighting.git" ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting \
 && git clone "https://github.com/zsh-users/zsh-completions" ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions \
 && git clone "https://github.com/supercrabtree/k" ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/k
ADD .zshrc /home/${USERNAME}
ENV SHELL /bin/zsh

#===>> bash setup
ADD .bashrc /home/${USERNAME}
ENV SHELL /bin/bash

CMD ["/bin/sh", "-c", "source /home/${USERNAME}/.bashrc"]
