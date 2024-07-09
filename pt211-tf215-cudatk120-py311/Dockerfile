#!/bin/bash

# Paperspace Dockerfile for Gradient base image
# Paperspace image is located in Dockerhub registry: paperspace/gradient_base

# ==================================================================
# Module list
# ------------------------------------------------------------------
# python                        3.11.6           (apt)
# pip3                          23.3.1           (apt)
# cuda toolkit                  12.0.0           (apt)
# cudnn                         8.9.7            (apt)
# jupyter_contrib_nbextensions  0.7.0            (pip)
# jupyterlab-git                0.43.0           (pip)


# ==================================================================
# Initial setup
# ------------------------------------------------------------------

# Ubuntu 22.04 as base image
FROM amd64/ubuntu:22.04
RUN yes| unminimize

# Set ENV variables
ENV LANG C.UTF-8
ENV SHELL=/bin/bash
ENV DEBIAN_FRONTEND=noninteractive

ENV APT_INSTALL="apt-get install -y --no-install-recommends"
ENV PIP_INSTALL="python3 -m pip --no-cache-dir install --upgrade"
ENV GIT_CLONE="git clone --depth 10"


# ==================================================================
# Tools
# ------------------------------------------------------------------

RUN apt-get update && \
    $APT_INSTALL \
    gcc \
    make \
    pkg-config \
    apt-transport-https \
    build-essential \
    apt-utils \
    ca-certificates \
    wget \
    rsync \
    git \
    vim \
    mlocate \
    libssl-dev \
    curl \
    openssh-client \
    unzip \
    unrar \
    zip \
    awscli \
    csvkit \
    emacs \
    joe \
    jq \
    dialog \
    man-db \
    manpages \
    manpages-dev \
    manpages-posix \
    manpages-posix-dev \
    nano \
    iputils-ping \
    sudo \
    ffmpeg \
    libsm6 \
    libxext6 \
    libboost-all-dev \
    gnupg \
    cifs-utils \
    zlib1g \
    software-properties-common

# ==================================================================
# Git-lfs
# ------------------------------------------------------------------

RUN curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash && \
    $APT_INSTALL git-lfs


# ==================================================================
# Python
# ------------------------------------------------------------------

#Based on https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa

# Adding repository for python3.11
RUN add-apt-repository ppa:deadsnakes/ppa -y && \
    $APT_INSTALL \
    python3.11 \
    python3.11-dev \
    python3.11-venv \
    python3-distutils-extra

# Add symlink so python and python3 commands use same python3.9 executable
RUN ln -s /usr/bin/python3.11 /usr/local/bin/python3 && \
    ln -s /usr/bin/python3.11 /usr/local/bin/python

# Installing pip
RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python3
ENV PATH=$PATH:/root/.local/bin

# ==================================================================
# Installing CUDA packages (CUDA Toolkit 12.0 and CUDNN 8.9.7)
# ------------------------------------------------------------------
RUN wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin && \
    mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600 && \
    wget https://developer.download.nvidia.com/compute/cuda/12.1.0/local_installers/cuda-repo-ubuntu2204-12-1-local_12.1.0-530.30.02-1_amd64.deb && \
    dpkg -i cuda-repo-ubuntu2204-12-1-local_12.1.0-530.30.02-1_amd64.deb && \
    cp /var/cuda-repo-ubuntu2204-12-1-local/cuda-*-keyring.gpg /usr/share/keyrings/ && \
    apt-get update && \
    $APT_INSTALL cuda && \
    rm cuda-repo-ubuntu2204-12-1-local_12.1.0-530.30.02-1_amd64.deb

# Installing CUDNN
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/3bf863cc.pub && \
    add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/ /" && \
    apt-get update && \
    $APT_INSTALL libcudnn8=8.9.7.29-1+cuda12.2  \
    libcudnn8-dev=8.9.7.29-1+cuda12.2


ENV PATH=$PATH:/usr/local/cuda/bin
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

# ==================================================================
# Additional Python Packages
# ------------------------------------------------------------------

RUN $PIP_INSTALL jupyterlab

# ==================================================================
# CMake
# ------------------------------------------------------------------

RUN $GIT_CLONE https://github.com/Kitware/CMake ~/cmake && \
    cd ~/cmake && \
    ./bootstrap && \
    make -j"$(nproc)" install


# ==================================================================
# Node.js and Jupyter Notebook Extensions
# ------------------------------------------------------------------

RUN curl -sL https://deb.nodesource.com/setup_20.x | bash  && \
    $APT_INSTALL nodejs  && \
    $PIP_INSTALL jupyter_contrib_nbextensions==0.7.0 jupyterlab-git==0.43.0 && \
    jupyter contrib nbextension install --user


# ==================================================================
# Startup
# ------------------------------------------------------------------

EXPOSE 8888 6006

CMD jupyter lab --allow-root --ip=0.0.0.0 --no-browser --ServerApp.trust_xheaders=True --ServerApp.disable_check_xsrf=False --ServerApp.allow_remote_access=True --ServerApp.allow_origin='*' --ServerApp.allow_credentials=True
