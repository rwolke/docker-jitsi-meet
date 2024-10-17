FROM docker.io/library/debian:bookworm-slim

ARG JITSI_RELEASE=stable
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2

COPY rootfs /

RUN dpkgArch="$(dpkg --print-architecture)"
RUN case "${dpkgArch##*-}" in \
        "amd64") TPL_ARCH=amd64; S6_ARCH=amd64 ;; \
        "arm64") TPL_ARCH=arm64; S6_ARCH=aarch64 ;; \
        "s390x") TPL_ARCH=s390x; S6_ARCH=s390x ;; \
        *) echo "unsupported architecture"; exit 1 ;; \
    esac
RUN apt-dpkg-wrap apt-get update
RUN apt-dpkg-wrap apt-get install -y apt-transport-https apt-utils ca-certificates gnupg wget curl
RUN wget -qO /usr/bin/tpl https://github.com/rwolke/tpl/releases/download/v0.0.1/tpl-linux-${TPL_ARCH}
# Workaround S6 bug when /bin is a symlink
RUN wget -qO /tmp/s6.tar.gz https://github.com/just-containers/s6-overlay/releases/download/v1.22.1.0/s6-overlay-${S6_ARCH}.tar.gz
RUN mkdir /tmp/s6
RUN tar xfz /tmp/s6.tar.gz -C /tmp/s6
RUN tar hxfz /tmp/s6.tar.gz -C /
RUN rm -f /usr/bin/execlineb
RUN cp /tmp/s6/bin/execlineb /usr/bin/
RUN rm -rf /tmp/s6*
RUN wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | gpg --dearmour > /etc/apt/trusted.gpg.d/jitsi.gpg
RUN echo "deb https://download.jitsi.org $JITSI_RELEASE/" > /etc/apt/sources.list.d/jitsi.list
RUN echo "deb http://ftp.debian.org/debian bookworm-backports main" > /etc/apt/sources.list.d/backports.list
RUN apt-dpkg-wrap apt-get update
RUN apt-dpkg-wrap apt-get dist-upgrade -y
RUN apt-cleanup
RUN chmod +x /usr/bin/tpl

RUN [ "$JITSI_RELEASE" = "unstable" ] && \
    apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get install -y jq procps curl vim iputils-ping net-tools && \
    apt-cleanup || \
    true

ENTRYPOINT [ "/init" ]
