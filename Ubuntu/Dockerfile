# syntax=docker/dockerfile:1
##
## Configuration
##
# set Ubuntu release version (latest, focal, bionic, ...)
ARG UBUNTU_VERSION=jammy
# set FPC version (format: d.d.d)
ARG FPC_VERSION=3.2.2


##
## Generate script to download official FPC installer
##
FROM ubuntu:${UBUNTU_VERSION} as dlscript
WORKDIR /tmp
# create script
RUN echo "#!/bin/sh" >> dl.sh \
    && echo "case \$TARGETPLATFORM in" >> dl.sh \
    && echo " "linux/amd64") FPC_ARCHITECTURE="x86_64-linux";;" >> dl.sh \
    && echo " "linux/arm64") FPC_ARCHITECTURE="aarch64-linux";;" >> dl.sh \
    && echo " #"linux/riscv64") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " #"linux/ppc64le") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " #"linux/s390x") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " # As of Ubuntu 19.10 i386 is no longer supported!" >> dl.sh \
    && echo " #"linux/386") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " #"linux/arm/v6") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " "linux/arm/v7") FPC_ARCHITECTURE="arm-linux";;" >> dl.sh \
    && echo " *) echo "No mapping for architecture \${TARGETPLATFORM} defined!"; exit;" >> dl.sh \
    && echo "esac" >> dl.sh \
    && echo "echo "Downloading FPC \${FPC_ARCHITECTURE} for \${TARGETPLATFORM} target."" >> dl.sh \
    && echo "# SourceForge is probably the worst download source you can imagine..." >> dl.sh \
    && echo "# wget --output-document="fpc-archive.tar" "https://downloads.sourceforge.net/project/freepascal/Linux/\${FPC_VERSION}/fpc-\${FPC_VERSION}.\${FPC_ARCHITECTURE}.tar"" >> dl.sh \
    && echo "wget --output-document="fpc-archive.tar" "https://mirror.freemirror.org/pub/fpc/dist/\${FPC_VERSION}/\${FPC_ARCHITECTURE}/fpc-\${FPC_VERSION}.\${FPC_ARCHITECTURE}.tar"" >> dl.sh


##
## Full installation via official FPC installer
##
# reuse previous argument value
ARG UBUNTU_VERSION
FROM ubuntu:${UBUNTU_VERSION} as fpc-full
WORKDIR /ws
# copy created download script
COPY --from=dlscript /tmp/dl.sh /tmp/dl.sh
# reuse previous argument value
ARG FPC_VERSION
# docker buildx variables
ARG TARGETPLATFORM
ARG BUILDPLATFORM
# update system and install dependencies, download and install FPC
RUN set -eux \
    && apt-get update \
    && apt-get install -y build-essential \
        wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && echo "Trying to build FPC minimal image for ${TARGETPLATFORM} on ${BUILDPLATFORM}." \
    && cd /tmp \
    # download
    && sh dl.sh \
    # unpack
    && tar -xvf fpc-archive.tar \
    && cd fpc-* \
    # install
    && ./install.sh \
    # cleanup
    && cd /ws \
    && rm -r /tmp/*
