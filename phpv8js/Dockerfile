FROM php:7-apache as v8builder
ENV V8_VERSION=6.4.388.18
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    curl \
    git \
    libglib2.0-dev \
    python && \
    cd /tmp && \
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git && \
    export PATH=`pwd`/depot_tools:"$PATH" && \
    fetch v8 && \
    cd v8 && \
    git checkout $V8_VERSION && \
    gclient sync && \
    tools/dev/v8gen.py -vv x64.release -- is_component_build=true && \
    ninja -C out.gn/x64.release/

FROM php:7-apache
RUN mkdir -p /opt/v8/lib && \
    mkdir -p /opt/v8/include
COPY --from=v8builder /tmp/v8/out.gn/x64.release/lib*.so /opt/v8/lib/
COPY --from=v8builder /tmp/v8/out.gn/x64.release/*_blob.bin /opt/v8/lib/
COPY --from=v8builder /tmp/v8/out.gn/x64.release/icudtl.dat /opt/v8/lib/
COPY --from=v8builder /tmp/v8/include /tmp/v8_include
RUN cp -R /tmp/v8_include/* /opt/v8/include/ && \
    apt-get update && \
    apt-get install -y \
    build-essential \
    git \
    patchelf && \
    for A in /opt/v8/lib/*.so; do patchelf --set-rpath '$ORIGIN' $A; done && \
    cd /tmp && \
    git clone https://github.com/phpv8/v8js.git && \
    cd v8js && \
    phpize && \
    ./configure --with-v8js=/opt/v8 LDFLAGS="-lstdc++" && \
    make && \
    make test && \
    make install && \
    docker-php-ext-enable v8js && \
    apt-get -y remove --purge \
    build-essential \
    git \
    patchelf && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*
