FROM thorenv/nvidia-ubuntu-16.04:av-base

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
