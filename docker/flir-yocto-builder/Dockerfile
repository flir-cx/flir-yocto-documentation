ARG FROM_TAG=14.04
FROM ubuntu:$FROM_TAG
LABEL maintainer="Richard Kjerstadius"
LABEL description="Container for Yocto development on FLIR camera platforms"

# Workaround for the fact that any ARG declared before FROM is outside of a
# build stage and therefore not visible inside the build stage.
# See https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact
# FWIW, buildah doesn't suffer from this limitation
ARG FROM_TAG

ENV COMMON_PKGS bash-completion \
                build-essential \
                ca-certificates \
                chrpath \
                diffstat \
                gawk \
                git-core \
                iproute2 \
                iputils-ping \
                libncurses-dev \
                libpng-dev \
                libssl-dev \
                less \
                locales \
                lzop \
                net-tools \
                openssh-client \
                python \
                rsync \
                socat \
                sudo \
                texinfo \
                tmux \
                tree \
                unzip \
                wget \
                nano \
                vim \
                jed \
                emacs-nox

ENV PKGS_1404 gcc-multilib

ENV PKGS_1604 cpio \
              debianutils \
              file \
              libegl1-mesa  \
              libsdl1.2-dev \
              libncurses-dev \
              pylint3 \
              python3 \
              python3-git \
              python3-jinja2 \
              python3-pexpect \
              python3-pip \
              xterm \
              xz-utils 

ENV PKGS_1804 default-jdk \ 
            docbook2x \
            fakechroot \
            fakeroot \
            g++-aarch64-linux-gnu \
            gcc-aarch64-linux-gnu \
            libarchive-dev  \
            libiberty-dev \
            libpam0g-dev \
            libselinux1-dev \
            libwayland-bin \
            libwayland-dev \
            libxml-simple-perl \
            qemu-user-static  \
            xxd \
            zlib1g-dev \
            libarchive-dev \
            libselinux1-dev \
            fakechroot \
            fakeroot\
            libiberty-dev \
            qemu-user-static\
            libxml-simple-perl \
            gcc-aarch64-linux-gnu \
            g++-aarch64-linux-gnu \
            docbook2x \
            sshpass

ENV PKGS_2004 liblz4-tool \
            zstd

ENV DEBIAN_FRONTEND noninteractive

ENV APT_CLEAN_CMD apt-get clean
ENV APT_INSTALL_CMD apt-get install -y --no-install-recommends
ENV APT_UPDATE_CMD apt-get update
# Remove Extended Security Maintenance source. It (sometimes) requires login
RUN rm -f /etc/apt/sources.list.d/ubuntu-esm-infra-trusty.list
RUN if [ "$FROM_TAG" = "20.04" ]; then \
        $APT_UPDATE_CMD \
        && $APT_INSTALL_CMD \
        $COMMON_PKGS \
        $PKGS_1604 \
        $PKGS_1804 \
        $PKGS_2004 \
        && $APT_CLEAN_CMD ; \
    elif [ "$FROM_TAG" = "18.04" ]; then \
        $APT_UPDATE_CMD \
        && $APT_INSTALL_CMD \
        $COMMON_PKGS \
        $PKGS_1604 \
        $PKGS_1804 \
        python-pip \
        && $APT_CLEAN_CMD ; \
    elif [ "$FROM_TAG" = "16.04" ]; then \
        $APT_UPDATE_CMD \
        && $APT_INSTALL_CMD \
        $COMMON_PKGS \
        $PKGS_1604 \
        $PKGS_1404 \
        && $APT_CLEAN_CMD ; \
    else \
        $APT_UPDATE_CMD \
        && $APT_INSTALL_CMD \
        $COMMON_PKGS \
        $PKGS_1404 \
        && $APT_CLEAN_CMD ; \
    fi

# Install repo
RUN wget https://storage.googleapis.com/git-repo-downloads/repo -O /usr/bin/repo && \
    chmod a+x /usr/bin/repo

RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

RUN echo "root:docker" | chpasswd

RUN rm -rf /lib/ld-linux-aarch64.so.1; ln -sf /usr/aarch64-linux-gnu/lib/ld-2.27.so /lib/ld-linux-aarch64.so.1

RUN wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz; \
 tar -C /usr/local -xzf go1.14.4.linux-amd64.tar.gz; \
    rm go1.14.4.linux-amd64.tar.gz;

# Install tools for python testing
RUN if [ "$FROM_TAG" = "18.04" ] || [ "$FROM_TAG" = "20.04" ]; then \
       pip install paramiko nose; \
    fi

ENV USERNAME yoctobuild

ARG UID=2112
ARG GID=2112

RUN groupadd -g $GID $USERNAME && \
    useradd --no-log-init -g $GID -u $UID -m $USERNAME \
    && adduser $USERNAME sudo \
    && echo '%sudo ALL=NOPASSWD: ALL' > /etc/sudoers.d/sudo-group \
    && chmod 0440 /etc/sudoers.d/sudo-group

#change default shell to bash
RUN chsh -s /bin/bash $USERNAME
RUN ln -sf /bin/bash /bin/sh

RUN echo 'PS1="\[\033[01;34m\]ubuntu:\W \[\033[00m\]$ "' >> /etc/bash.bashrc

USER $USERNAME
RUN mkdir /home/yoctobuild/.ssh

CMD ssh-agent /bin/bash
