ARG NGINX_VERSION=1.25.1
ARG PCRE_VERSION=10.42
ARG ZLIB_VERSION=1.3.1
ARG OPENSSL_VERSION=1.1.1v

ARG S6_OVERLAY_VERSION=3.1.4.1

ARG RTMP_VERSION=1.2.2

# Downloads and builds are separated into individual stages to
# allow for parallel building of dependencies while building
# nginx. Downloads are also not unified into one image as a change
# in one version would require a redownload and rebuild of all
# dependencies.

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
        linux-headers && \
    echo "Create build and source directories" && \
    mkdir -p /build /source

FROM build AS download-pcre2
ARG PCRE_VERSION
WORKDIR /source
RUN wget github.com/PCRE2Project/pcre2/releases/download/pcre2-$PCRE_VERSION/pcre2-$PCRE_VERSION.tar.gz
RUN tar -zxf pcre2-$PCRE_VERSION.tar.gz -C .
RUN rm -f pcre2-$PCRE_VERSION.tar.gz
RUN mv pcre2-$PCRE_VERSION/ ./pcre2

FROM download-pcre2 AS build-pcre2
WORKDIR /source/pcre2
RUN ./configure --prefix=/build/pcre2/
RUN make


FROM build AS download-zlib
ARG ZLIB_VERSION
WORKDIR /source
RUN wget https://zlib.net/zlib-$ZLIB_VERSION.tar.gz
RUN tar -zxf zlib-$ZLIB_VERSION.tar.gz -C .
RUN rm -f zlib-$ZLIB_VERSION.tar.gz
RUN mv zlib-$ZLIB_VERSION/ ./zlib

FROM download-zlib AS build-zlib
WORKDIR /source/zlib
RUN ./configure --prefix=/build/zlib/
RUN make



FROM build as download-openssl
ARG OPENSSL_VERSION
WORKDIR /source
RUN wget https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz
RUN tar -zxf openssl-$OPENSSL_VERSION.tar.gz -C .
RUN rm -f openssl-$OPENSSL_VERSION.tar.gz
RUN mv openssl-$OPENSSL_VERSION/ ./openssl

FROM download-openssl AS build-openssl
WORKDIR /source/openssl
RUN ./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl/ssl \
    && make \
    && make install_sw

FROM build AS build-nginx
ARG NGINX_VERSION
COPY --from=download-pcre2 /source/pcre2 /source/pcre2
COPY --from=download-zlib /source/zlib /source/zlib
COPY --from=download-openssl /source/openssl /source/openssl

WORKDIR /source

RUN wget https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz
RUN tar zxf nginx-$NGINX_VERSION.tar.gz
RUN rm -f nginx-$NGINX_VERSION.tar.gz

WORKDIR /source/nginx-$NGINX_VERSION
RUN ./configure \
        --prefix=/usr/local/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --with-pcre=/source/pcre2 \
        --with-pcre-jit \
        --with-zlib=/source/zlib \
        --with-openssl=/source/openssl \
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


# Download and extract s6-overlay

FROM alpine:latest AS s6-base
ARG S6_OVERLAY_VERSION
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz /tmp
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-x86_64.tar.xz /tmp
RUN tar xvf /tmp/s6-overlay-noarch.tar.xz -C /
RUN tar xvf /tmp/s6-overlay-x86_64.tar.xz -C /

# Start the final image

FROM s6-base AS final
COPY --from=build-nginx /usr/local/nginx /usr/local/nginx
COPY --from=build-nginx /etc/nginx /etc/nginx
    
# Copy OpenSSL from build-openssl image
COPY --from=build-openssl /usr/local/openssl /usr/local/openssl
ENV PATH="/usr/local/openssl/bin:${PATH}"

# Update library configuration for Alpine Linux
RUN echo "/usr/local/openssl/lib" > /etc/ld-musl-x86_64.path
    
# Create nginx user and group
#RUN adduser -D -H -u 1000 -s /bin/sh -G users nginx

# Create config directory and set ownership
#RUN mkdir /config && chown nginx:users /config

# Set the user to run the container as
#USER nginx

ADD preconfig/ /

ENTRYPOINT [ "/init" ]

EXPOSE 80/tcp 443/tcp 80/udp 443/udp

VOLUME [ "/config" ]