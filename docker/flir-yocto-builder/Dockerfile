ARG FROM_TAG=16.04
FROM ubuntu:$FROM_TAG
LABEL maintainer="Richard Kjerstadius"
LABEL description="Container for Yocto development on FLIR camera platforms"

ADD http://se-arn-dev5.zone2.flir.net/pingu/ubuntusources_swe.list /etc/apt/sources.list

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
                gcc-multilib \
                git-core \
                iproute2 \
                iputils-ping \
                libncurses-dev \
                libpng-dev \
                libssl-dev \
                locales \
                lzop \
                net-tools \
                openssh-client \
                python \
                socat \
                texinfo \
                tmux \
                unzip \
                wget
ENV PKGS_1604 cpio \
              debianutils \
              file \
              libegl1-mesa  \
              libsdl1.2-dev \
              pylint3 \
              python3 \
              python3-git \
              python3-jinja2 \
              python3-pexpect \
              python3-pip \
              repo \
              xterm \
              xz-utils 
ENV PKGS_1804 xxd 

ENV APT_CLEAN_CMD apt-get clean
ENV APT_INSTALL_CMD apt-get install -y --no-install-recommends
ENV APT_UPDATE_CMD apt-get update
# Remove Extended Security Maintenance source. It (sometimes) requires login
RUN rm -f /etc/apt/sources.list.d/ubuntu-esm-infra-trusty.list
RUN if [ "$FROM_TAG" = "18.04" ]; then \
        $APT_UPDATE_CMD \
        && $APT_INSTALL_CMD \
        $COMMON_PKGS \
        $PKGS_1604 \
        $PKGS_1804 \
        && $APT_CLEAN_CMD ; \
    elif [ "$FROM_TAG" = "16.04" ]; then \
        $APT_UPDATE_CMD \
        && $APT_INSTALL_CMD \
        $COMMON_PKGS \
        $PKGS_1604 \
        && $APT_CLEAN_CMD ; \
    else \
        $APT_UPDATE_CMD \
        && $APT_INSTALL_CMD \
        $COMMON_PKGS \
        && $APT_CLEAN_CMD ; \
    fi

# Install FLIR certificates
RUN cd /usr/local/share/ca-certificates && \
    wget -O rootca.crt http://se-arn-ubtmirr.zone2.flir.net/esw/certs/rootca.cer && \
    wget -O flir-orca.crt http://se-arn-ubtmirr.zone2.flir.net/esw/certs/flir-orca.cer && \
    /usr/sbin/update-ca-certificates

# Get Perforce client CLI
RUN wget http://www.perforce.com/downloads/perforce/r19.1/bin.linux26x86_64/helix-core-server.tgz && \
    mkdir perforce-bin && \
    tar xvf helix-core-server.tgz -C perforce-bin && \
    cp perforce-bin/p4 /usr/bin/. && \
    chmod +x /usr/bin/p4 && \
    rm -rf helix-core-server.tgx perforce-bin

RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

RUN echo "root:docker" | chpasswd

ENV USERNAME yoctobuild

ARG UID=2112
ARG GID=2112

RUN groupadd -g $GID $USERNAME && \
    useradd --no-log-init -g $GID -u $UID -m $USERNAME

RUN sed -i 's/^PS1.*/PS1=\"\\[\\033\[01;34m\\]ubuntu:\\W \\[\\033\[00m\\]$ \"/g' /etc/bash.bashrc
USER $USERNAME

CMD ssh-agent /bin/bash
