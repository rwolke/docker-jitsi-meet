FROM docker.io/library/debian:bullseye-slim

ARG JITSI_RELEASE=stable
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2

COPY rootfs /

RUN \
    dpkgArch="$(dpkg --print-architecture)" && \
    case "${dpkgArch##*-}" in \
        "amd64") TPL_ARCH=amd64; S6_ARCH=x86_64 ;; \
        "arm64") TPL_ARCH=arm64; S6_ARCH=aarch64 ;; \
        "s390x") TPL_ARCH=s390x; S6_ARCH=s390x ;; \
        *) echo "unsupported architecture"; exit 1 ;; \
    esac && \
    echo "ARCH: $TPL_ARCH"
RUN echo "ARCH: $TPL_ARCH" && \
    apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get install -y apt-transport-https apt-utils ca-certificates gnupg wget xz-utils && \
    wget -qO /usr/bin/tpl https://github.com/rwolke/tpl/releases/download/v0.0.1/tpl-linux-${TPL_ARCH} && \
    wget -qO - https://github.com/just-containers/s6-overlay/releases/download/v3.1.6.2/s6-overlay-${S6_ARCH}.tar.xz | unxz -c | tar xf - -C /
    wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | gpg --dearmour > /etc/apt/trusted.gpg.d/jitsi.gpg

RUN echo "deb https://download.jitsi.org $JITSI_RELEASE/" > /etc/apt/sources.list.d/jitsi.list && \
    echo "deb http://ftp.debian.org/debian bullseye-backports main" > /etc/apt/sources.list.d/backports.list && \
    apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get dist-upgrade -y && \
    apt-cleanup && \
    chmod +x /usr/bin/tpl

RUN [ "$JITSI_RELEASE" = "unstable" ] && \
    apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get install -y jq procps curl vim iputils-ping net-tools && \
    apt-cleanup || \
    true

ENTRYPOINT [ "/init" ]
