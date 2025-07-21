FROM ubuntu:22.04

# 1. Cài gói cần thiết
RUN apt update && apt install -y --no-install-recommends \
    build-essential git curl wget ca-certificates \
    libtool autoconf automake pkg-config libssl-dev \
    libev-dev python3 python3-pip cmake \
    && rm -rf /var/lib/apt/lists/*

# 2. Build và cài wolfSSL
RUN git config --global http.postBuffer 524288000 && \
    git clone https://github.com/wolfSSL/wolfssl.git && \
    cd wolfssl && \
    ./autogen.sh && \
    ./configure --enable-tls13 --enable-all --enable-quic && \
    make -j$(nproc) && make install && \
    cd .. && rm -rf wolfssl


# 3. Export path cho pkg-config
ENV PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
ENV LD_LIBRARY_PATH=/usr/local/lib

# 4. Build và cài nghttp3 (dùng đúng commit tương thích sfparse cũ)
RUN git clone --recursive https://github.com/ngtcp2/nghttp3 && \
    cd nghttp3 && \
    git checkout a13d679a0c727fcf5e8cde4e6a8f5f292b0c06b4 && \
    git submodule update --init --recursive && \
    autoreconf -fi && \
    ./configure --prefix=/usr/local \
        CFLAGS="-I/usr/local/include" \
        LDFLAGS="-L/usr/local/lib" && \
    make -j$(nproc) && make install && \
    cd .. && rm -rf nghttp3

# 5. Build và cài ngtcp2 (với wolfSSL)
RUN git clone https://github.com/ngtcp2/ngtcp2 && \
    cd ngtcp2 && \
    autoreconf -fi && \
    ./configure --prefix=/usr/local \
        --enable-lib-only \
        --with-wolfssl && \
    make -j$(nproc) && make install && \
    cd .. && rm -rf ngtcp2

# 6. Build curl với HTTP/3 + wolfSSL
RUN wget https://curl.se/download/curl-8.7.1.tar.gz && \
    tar -xzf curl-8.7.1.tar.gz && \
    cd curl-8.7.1 && \
    ./configure --with-wolfssl \
                --with-nghttp3 \
                --with-ngtcp2 \
                --enable-alt-svc \
                --enable-http3 \
                --enable-debug && \
    make -j$(nproc) && make install && \
    cd .. && rm -rf curl-8.7.1 curl-8.7.1.tar.gz

# 7. Khai báo thư viện runtime
RUN echo "/usr/local/lib" > /etc/ld.so.conf.d/local.conf && ldconfig

# 8. Copy code vào container
WORKDIR /app
COPY . /app

