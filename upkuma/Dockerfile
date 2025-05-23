# syntax=mirror.gcr.io/docker/dockerfile:1

FROM mirror.gcr.io/bitnami/minideb:buster AS graftcp-builder
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN mkdir -p '/etc/dpkg/dpkg.cfg.d' '/etc/apt/apt.conf.d' \
    && echo 'force-unsafe-io' > '/etc/dpkg/dpkg.cfg.d/docker-apt-speedup' \
    && echo 'Acquire::Languages "none";' > '/etc/apt/apt.conf.d/docker-no-languages' \
    && echo -e 'Acquire::GzipIndexes "true";\nAcquire::CompressionTypes::Order:: "gz";' > '/etc/apt/apt.conf.d/docker-gzip-indexes' \
    && apt-get update -qq \
    && apt-get -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false purge \
        libpython-stdlib libpython2-stdlib libpython2.7-minimal libpython2.7-stdlib python python-minimal python2 python2-minimal python2.7 python2.7-minimal \
        mercurial mercurial-common \
        mime-support \
    && apt-get -y --no-install-recommends install \
        binutils build-essential coreutils file netbase \
        ca-certificates \
        gzip tar \
        ucf \
    && apt-get full-upgrade -y \
    && apt-get clean \
    && rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/*

ARG TARGETARCH
ENV GOLANG_VERSION=1.22.5 \
    PATH="/usr/local/go/bin:${PATH}"
ADD --link "https://go.dev/dl/go${GOLANG_VERSION}.linux-${TARGETARCH}.tar.gz" /go.linux.tar.gz
RUN rm -rf /usr/local/go \
    && tar -C /usr/local/ -xzf /go.linux.tar.gz

WORKDIR /emptydir/
WORKDIR /git/graftcp/
ADD --link "https://github.com/hmgle/graftcp.git" ./
ENV CFLAGS="-O2 -pipe -D_FORTIFY_SOURCE=2 -fexceptions -fstack-clash-protection -fstack-protector-strong -g -grecord-gcc-switches -Wl,-z,noexecstack,-z,relro,-z,now,-z,defs" \
  CXXFLAGS="-O2 -pipe -D_FORTIFY_SOURCE=2 -fexceptions -fstack-clash-protection -fstack-protector-strong -g -grecord-gcc-switches -Wl,-z,noexecstack,-z,relro,-z,now,-z,defs"
RUN go env -w GOFLAGS="$GOFLAGS -buildmode=pie" \
    && go env -w GO111MODULE=on \
    && make \
    && make install \
    && rm -rf /git/graftcp/ /go/ /root/.cache/


FROM mirror.gcr.io/louislam/uptime-kuma:1 AS builder
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN mkdir -p '/etc/dpkg/dpkg.cfg.d' '/etc/apt/apt.conf.d' \
    && echo 'force-unsafe-io' > '/etc/dpkg/dpkg.cfg.d/docker-apt-speedup' \
    && echo 'Acquire::Languages "none";' > '/etc/apt/apt.conf.d/docker-no-languages' \
    && echo -e 'Acquire::GzipIndexes "true";\nAcquire::CompressionTypes::Order:: "gz";' > '/etc/apt/apt.conf.d/docker-gzip-indexes' \
    && apt-get update -qq && apt-get full-upgrade -y \
    && apt-get -y --no-install-recommends install \
        ca-certificates \
    && apt-get -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false purge \
    && apt-get clean \
    && rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/*


FROM scratch
COPY --link --from=builder                     /  /
COPY --link --from=graftcp-builder --chmod=755 /usr/local/bin/graftcp /usr/local/bin/graftcp-local /usr/local/bin/mgraftcp \
                                               /usr/local/bin/
ENV TZ=Asia/Taipei
ENV UPTIME_KUMA_IS_CONTAINER=1

ENTRYPOINT [ "/usr/local/bin/mgraftcp" ]
CMD [ "extra/entrypoint.sh", "node", "server/server.js" ]

WORKDIR /app/
VOLUME ["/app/data"]
HEALTHCHECK --interval=60s --timeout=30s --start-period=180s --retries=5 CMD extra/healthcheck
