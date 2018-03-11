FROM php:7-apache
ENV V8_VERSION=6.4.388.18
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    curl \
    git \
    libglib2.0-dev \
    patchelf \
    python && \
    cd /tmp && \
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git && \
    export PATH=`pwd`/depot_tools:"$PATH" && \
    fetch v8 && \
    cd v8 && \
    git checkout $V8_VERSION && \
    gclient sync && \
    tools/dev/v8gen.py -vv x64.release -- is_component_build=true && \
    ninja -C out.gn/x64.release/ && \
    mkdir -p /opt/v8/lib && \
    mkdir -p /opt/v8/include && \
    cp out.gn/x64.release/lib*.so out.gn/x64.release/*_blob.bin \
        out.gn/x64.release/icudtl.dat /opt/v8/lib/ && \
    cp -R include/* /opt/v8/include/ && \
    rm -rf /tmp/v8 && \
    rm -rf /tmp/depot_tools && \
    for A in /opt/v8/lib/*.so; do patchelf --set-rpath '$ORIGIN' $A; done && \
    apt-get remove --purge \
    build-essential \
    curl \
    git \
    libglib2.0-dev \
    patchelf \
    python && \
    rm -rf /var/lib/apt/lists/*
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    git && \
    cd /tmp && \
    git clone https://github.com/phpv8/v8js.git && \
    cd v8js && \
    phpize && \
    ./configure --with-v8js=/opt/v8 LDFLAGS="-lstdc++" && \
    make && \
    make test && \
    make install && \
    docker-php-ext-enable v8js && \
    apt-get remove --purge \
    build-essential \
    git

