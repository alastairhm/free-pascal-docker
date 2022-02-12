# syntax=docker/dockerfile:1
##
## Configuration
##
# set Alpine Linux version (latest or concrete version)
ARG ALPINE_VERSION=3.15
# set FPC version (format: d.d.d)
ARG FPC_VERSION=3.2.0
# define if documentation, examples, etc should be installed
ARG DO_FULL_INSTALLATION=0


FROM alpine:$ALPINE_VERSION as gcc
# update system and install gcc
RUN apk update \
    && apk upgrade \
    && apk add gcc \
    # get path and copy gcc library
    && gcc -print-libgcc-file-name > /tmp/lib.path \
    && cat /tmp/lib.path | xargs cp -t /tmp


##
## Generate script to download official FPC installer
##
# reuse previous argument value
ARG ALPINE_VERSION
FROM alpine:$ALPINE_VERSION as dlscript
WORKDIR /tmp
# create script
RUN echo "#!/bin/sh" >> dl.sh \
    && echo "case \$TARGETPLATFORM in" >> dl.sh \
    && echo " "linux/amd64") FPC_ARCHITECTURE="x86_64-linux";;" >> dl.sh \
    && echo " "linux/arm64") FPC_ARCHITECTURE="aarch64-linux";;" >> dl.sh \
    && echo " #"linux/riscv64") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " #"linux/ppc64le") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " #"linux/s390x") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " "linux/386") FPC_ARCHITECTURE="i386-linux";;" >> dl.sh \
    && echo " #"linux/arm/v6") FPC_ARCHITECTURE="";;" >> dl.sh \
    && echo " "linux/arm/v7") FPC_ARCHITECTURE="arm-linux";;" >> dl.sh \
    && echo " *) echo "No mapping for architecture \${TARGETPLATFORM} defined!"; exit;" >> dl.sh \
    && echo "esac" >> dl.sh \
    && echo "echo "Downloading FPC \${FPC_ARCHITECTURE} for \${TARGETPLATFORM} target."" >> dl.sh \
    && echo "# SourceForge is probably the worst download source you can imagine..." >> dl.sh \
    && echo "# wget --output-document="fpc-archive.tar" "https://downloads.sourceforge.net/project/freepascal/Linux/\${FPC_VERSION}/fpc-\${FPC_VERSION}.\${FPC_ARCHITECTURE}.tar"" >> dl.sh \
    && echo "# Packages do not follow the same naming scheme..." >> dl.sh \
    && echo "if [ "\${FPC_ARCHITECTURE}" = "x86_64-linux" ]; then" >> dl.sh \
    && echo " wget --output-document="fpc-archive.tar" "https://mirror.freemirror.org/pub/fpc/dist/\${FPC_VERSION}/\${FPC_ARCHITECTURE}/fpc-\${FPC_VERSION}-\${FPC_ARCHITECTURE}.tar"" >> dl.sh \
    && echo "else" >> dl.sh \
    && echo " wget --output-document="fpc-archive.tar" "https://mirror.freemirror.org/pub/fpc/dist/\${FPC_VERSION}/\${FPC_ARCHITECTURE}/fpc-\${FPC_VERSION}.\${FPC_ARCHITECTURE}.tar"" >> dl.sh \
    && echo "fi" >> dl.sh


##
## Setup installation environment
##
# reuse previous argument value
ARG ALPINE_VERSION
FROM alpine:$ALPINE_VERSION as install-env
WORKDIR /tmp
# copy gcc binary and libgcc library information
COPY --from=gcc /usr/bin/gcc /usr/bin/gcc
COPY --from=gcc /tmp /tmp
# update system, install needed dependencies and store library path
RUN apk update \
    && apk upgrade \
    && apk add binutils \
        # light weight glibc compatibility layer that creates symlinks for libc, libpthread, etc
        libc6-compat \
        # GNU make utility
        make \
        # musl C library implementation (libc, libpthread, etc)
        musl-dev \
    # create path and copy gcc library to it
    && echo "Using following path for gcc library:" \
    && cat lib.path \
    && cat lib.path | xargs -I path mkdir -p path \
    && cat lib.path | xargs -I path mv libgcc.a path


##
## Minimal installation via official FPC installer for compiling trunk
##
FROM install-env as fpc-minimal
WORKDIR /ws
# copy created download script
COPY --from=dlscript /tmp/dl.sh /tmp/dl.sh
# reuse previous argument value
ARG FPC_VERSION
# docker buildx variables
ARG TARGETPLATFORM
ARG BUILDPLATFORM
# download and extract installer, install and remove temporary files
RUN set -eux \
    && echo "Trying to build FPC minimal image for ${TARGETPLATFORM} on ${BUILDPLATFORM}." \
    && cd /tmp \
    # download
    && sh dl.sh \
    # unpack
    && tar -xvf fpc-archive.tar \
    && cd fpc-* \
    # install without documentation, examples, etc
    && echo -e '/usr\nN\nN\nN\n' | sh ./install.sh \
    # remove unneeded units and packages
    && find /usr/lib/fpc/${FPC_VERSION}/units/*/ -type d -mindepth 1 -maxdepth 1 \
        -not -name 'rtl' \
        -exec rm -r {} \; \
    # cleanup
    && cd /ws \
    && rm -r /tmp/*
CMD ["make", "fullcycle", "-j$(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}')"]


##
## Slim installation via official FPC installer
##
FROM install-env as fpc-slim
WORKDIR /ws
# copy created download script
COPY --from=dlscript /tmp/dl.sh /tmp/dl.sh
# reuse previous arguments values
ARG FPC_VERSION
# docker buildx variables
ARG TARGETPLATFORM
ARG BUILDPLATFORM
# download and extract installer, install and remove temporary files
RUN set -eux \
    && echo "Trying to build FPC slim image for ${TARGETPLATFORM} on ${BUILDPLATFORM}." \
    && cd /tmp \
    # download
    && sh dl.sh \
    # unpack
    && tar -xvf fpc-archive.tar \
    && cd fpc-* \
    # install without documentation, examples, etc
    && echo -e '/usr\nN\nN\nN\n' | sh ./install.sh \
    # remove unneeded units and packages
    && find /usr/lib/fpc/${FPC_VERSION}/units/*/ -type d -mindepth 1 -maxdepth 1 \
        -not -name 'fcl-base' \
        -not -name 'rtl' \
        -not -name 'rtl-console' \
        -not -name 'rtl-extra' \
        -not -name 'rtl-generics' \
        -not -name 'rtl-objpas' \
        -not -name 'rtl-unicode' \
        -exec rm -r {} \; \
    # cleanup
    && cd /ws \
    && rm -r /tmp/*


##
## Full installation via official FPC installer
##
FROM install-env as fpc-full
WORKDIR /ws
# copy created download script
COPY --from=dlscript /tmp/dl.sh /tmp/dl.sh
# reuse previous arguments values
ARG FPC_VERSION
ARG DO_FULL_INSTALLATION
# docker buildx variables
ARG TARGETPLATFORM
ARG BUILDPLATFORM
# download and extract installer, install and remove temporary files
RUN set -eux \
    && echo "Trying to build FPC full image for ${TARGETPLATFORM} on ${BUILDPLATFORM}." \
    && cd /tmp \
    # download
    && sh dl.sh \
    # unpack
    && tar -xvf fpc-archive.tar \
    && cd fpc-* \
    # install with or without documentation, examples, etc
    && test ${DO_FULL_INSTALLATION} -ge 1 && sh ./install.sh || echo -e '/usr\nN\nN\nN\n' | sh ./install.sh \
    # cleanup
    && cd /ws \
    && rm -r /tmp/*
