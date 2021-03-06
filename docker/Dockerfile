FROM jedisct1/base-dev-rust-nightly:62eb6056d
MAINTAINER Frank Denis
ENV SERIAL 2

ENV DEBIAN_FRONTEND noninteractive

ENV BUILD_DEPS \
    autoconf \
    automake \
    libtool \
    file \
    gcc \
    g++ \
    git \
    libc-dev \
    make \
    pkg-config

RUN set -x && \
    apt-get install -y \
        $BUILD_DEPS \
        libsnappy-dev \
        --no-install-recommends && \
    apt-get clean && \
    rm -fr /tmp/* /var/tmp/*

ENV LIBRESSL_VERSION 2.5.1
ENV LIBRESSL_SHA256 f71ae0a824b78fb1a47ffa23c9c26e9d96c5c9b29234eacedce6b4c7740287cd
ENV LIBRESSL_DOWNLOAD_URL http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-${LIBRESSL_VERSION}.tar.gz

RUN set -x && \
    mkdir -p /tmp/src && \
    cd /tmp/src && \
    curl -sSL $LIBRESSL_DOWNLOAD_URL -o libressl.tar.gz && \
    echo "${LIBRESSL_SHA256} *libressl.tar.gz" | sha256sum -c - && \
    tar xzf libressl.tar.gz && \
    rm -f libressl.tar.gz && \
    cd libressl-${LIBRESSL_VERSION} && \
    ./configure --disable-shared --with-pic --disable-dependency-tracking --prefix=/opt/libressl && \
    make check && make install && \
    rm -fr /opt/libressl/share/man && \
    echo /opt/libressl/lib > /etc/ld.so.conf.d/libressl.conf && ldconfig && \
    rm -fr /tmp/*

ENV OPENSSL_LIB_DIR=/opt/libressl/lib
ENV OPENSSL_INCLUDE_DIR=/opt/libressl/include

RUN set -x && \
    cd /tmp && \
    git clone https://github.com/jedisct1/flowgger.git && \
    cd flowgger && \
    cargo build --release --features='coroutines ecdh kafka' && \
    mkdir -p /opt/flowgger/etc /opt/flowgger/bin && \
    strip target/release/flowgger && \
    mv target/release/flowgger /opt/flowgger/bin/ && \
    rm -fr /tmp/flowgger

COPY flowgger.sh /etc/service/flowgger/run

EXPOSE 6514

ENTRYPOINT ["/sbin/my_init"]
