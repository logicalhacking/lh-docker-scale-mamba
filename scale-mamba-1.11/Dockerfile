FROM registry.hub.docker.com/library/debian:testing-slim

ARG USERNAME=lh
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ENV LD_LIBRARY_PATH=/opt/local/lib/:${LD_LIBRARY_PATH}
ENV LIBRARY_PATH=/opt/local/lib/:${LIBRARY_PATH}
ENV C_INCLUDE_PATH=/opt/local/include/:${C_INCLUDE_PATH}
ENV CPLUS_INCLUDE_PATH=/opt/local/include/:${CPLUS_INCLUDE_PATH}
ENV PATH=/opt/local/bin:/opt/scale-mamba/:${PATH}
ENV SCALE_MAMBA=/opt/scale-mamba/

# Configure apt and install packages
RUN mkdir -p /usr/share/man/man1
RUN apt-get update \
    #
    # Upgrade base image to ensure that all security updates are installed
    # && apt-get -y dist-upgrade \
    #
    # Ensure availability of common tools
    && apt-get -y install libssl-dev curl git openssh-client less iproute2 procps lsb-release \
    #
    # Install required develeopment tools (e.g., g++, python 2)
    && apt-get -y install  m4 g++ rustc yasm git unzip gcc libssl-dev make libcrypto++-dev python \
    #
    # Install MPIR 3.0.0
    && mkdir -p /opt/src/ \
    && cd /opt/src && curl http://mpir.org/mpir-3.0.0.tar.bz2 | tar xjf - \
    && cd /opt/src/mpir-3.0.0 && ./configure --enable-cxx --prefix=/opt/local && make -j 4 && make -j 4 check && make -j 4 install \
    # 
    # Install Cryptopp 820
    && mkdir -p /opt/src/ && cd /opt/src \
       && LD_LIBRARY_PATH= curl -O https://www.cryptopp.com/cryptopp820.zip && mkdir cryptopp820 \
    && cd /opt/src/cryptopp820 && unzip  ../cryptopp820.zip && rm ../cryptopp820.zip && make -j 4 && make -j 4 libcryptopp.so && PREFIX=/opt/local make -j 4 install \
    #
    # Requirements for remote sharing 
    && apt-get -y install libicu[0-9][0-9] libkrb5-3 zlib1g gnome-keyring libsecret-1-0 desktop-file-utils x11-utils openssl \
    # Install SCALE-MAMBA
    && LD_LIBRARY_PATH= git clone https://github.com/KULeuven-COSIC/SCALE-MAMBA.git /opt/src/SCALE-MAMBA \
    && cd /opt/src/SCALE-MAMBA \
       # Install SCALE-MAMBA version 1.11 (git commit 8123e690f622a79f6ed1199fe1be7db3cd23699a)
       && git checkout 8123e690f622a79f6ed1199fe1be7db3cd23699a \
       && cp CONFIG CONFIG.mine && echo ROOT=`pwd` >> CONFIG.mine \
       && make -j 4 progs \
       # install 
       && mkdir -p /opt/scale-mamba \
       && cp PlayerBinary.x /opt/scale-mamba/ \
       && cp SetupBinary.x /opt/scale-mamba/ \
       && cp scasm /opt/scale-mamba/ \
       && cp src/libMPC.a /opt/scale-mamba/ \
       && cp compile* /opt/scale-mamba/ \
       && cp -r Compiler /opt/scale-mamba/ \
    #
    # Create a non-root user to use if preferred 
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -ms /bin/bash -o --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# USER $USERNAME
ENV HOME /home/$USERNAME
WORKDIR /opt/scale-mamba

