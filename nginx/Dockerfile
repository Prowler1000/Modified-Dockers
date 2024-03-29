ARG NGINX_VERSION=1.25.1
ARG PCRE_VERSION=10.42
ARG ZLIB_VERSION=1.3.1
ARG OPENSSL_VERSION=1.1.1v

ARG S6_OVERLAY_VERSION=3.1.4.1

ARG RTMP_VERSION=1.2.2

#WORKDIR /prereqs/boringssl
#RUN \
#    echo "Download and configure BoringSSL" && \
#    git clone https://boringssl.googlesource.com/boringssl . && \
#    mkdir build && \
#    cd build && \
#    cmake -DCMAKE_BUILD_TYPE=Release .. && \
#    make

FROM alpine:latest AS build

RUN \
    echo "Update and install dependencies" && \
    apk update && \
    apk add --no-cache \
        git \
        make \
        cmake \
        wget \
        build-base \
        go \
        perl \
        linux-headers

FROM build AS build-pcre2
ARG PCRE_VERSION
WORKDIR /build

RUN wget github.com/PCRE2Project/pcre2/releases/download/pcre2-$PCRE_VERSION/pcre2-$PCRE_VERSION.tar.gz
RUN tar -zxf pcre2-$PCRE_VERSION.tar.gz -C .
RUN rm -f pcre2-$PCRE_VERSION.tar.gz
RUN mv pcre2-$PCRE_VERSION/ ./pcre2

FROM build AS build-zlib
ARG ZLIB_VERSION
WORKDIR /build
RUN wget https://zlib.net/zlib-$ZLIB_VERSION.tar.gz
RUN tar -zxf zlib-$ZLIB_VERSION.tar.gz -C .
RUN rm -f zlib-$ZLIB_VERSION.tar.gz
RUN mv zlib-$ZLIB_VERSION/ ./zlib

FROM build AS build-openssl
ARG OPENSSL_VERSION
WORKDIR /build
RUN wget https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz
RUN tar -zxf openssl-$OPENSSL_VERSION.tar.gz -C .
RUN rm -f openssl-$OPENSSL_VERSION.tar.gz
RUN mv openssl-$OPENSSL_VERSION/ ./openssl


# Don't forget to add the RTMP module

FROM build AS build-nginx
ARG NGINX_VERSION
COPY --from=build-pcre2 /build/pcre2 /build/pcre2
COPY --from=build-zlib /build/zlib /build/zlib
COPY --from=build-openssl /build/openssl /build/openssl

WORKDIR /build

RUN wget https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz
RUN tar zxf nginx-$NGINX_VERSION.tar.gz
RUN rm -f nginx-$NGINX_VERSION.tar.gz

WORKDIR /build/nginx-$NGINX_VERSION
RUN ./configure \
        --prefix=/usr/local/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --with-pcre=/build/pcre2 \
        --with-pcre-jit \
        --with-zlib=/build/zlib \
        --with-openssl=/build/openssl \
        --with-http_ssl_module \
        --with-http_v2_module \
        --with-stream \
        --with-stream_ssl_module \
        --with-threads \
        --with-debug \
        --with-http_v3_module \
        --with-cc-opt="-I../boringssl/include" \
        --with-ld-opt="-L../boringssl/build/ssl \
                       -L../boringssl/build/crypto"
RUN make
RUN make install

FROM alpine:latest AS s6-base
ARG S6_OVERLAY_VERSION
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz /tmp
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-x86_64.tar.xz /tmp
RUN tar xvf /tmp/s6-overlay-noarch.tar.xz -C /
RUN tar xvf /tmp/s6-overlay-x86_64.tar.xz -C /

FROM s6-base AS s6-nginx
COPY --from=build-nginx /usr/local/nginx /usr/local/nginx
COPY --from=build-nginx /etc/nginx /etc/nginx

ADD preconfig/ /

ENTRYPOINT [ "/init" ]

EXPOSE 80/tcp 443/tcp 80/udp 443/udp

VOLUME [ "/config" ]