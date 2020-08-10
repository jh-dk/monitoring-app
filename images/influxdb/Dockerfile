FROM quay.io/gravitational/debian-grande:stretch AS downloader
ARG INFLUXDB_VERSION

ENV DEBIAN_FRONTEND=noninteractive \
    TERM=xterm \
    INFLUXDB_HOME=/influxdb \
    PATH=$INFLUXDB_HOME:$PATH

RUN set -ex && apt-get update && \
    apt-get install --yes --no-install-recommends curl tar && \
    curl -sSL https://dl.influxdata.com/influxdb/releases/influxdb-${INFLUXDB_VERSION}-static_linux_amd64.tar.gz -o /influxdb.tar.gz && \
    tar xzf /influxdb.tar.gz --strip-components=2

FROM quay.io/gravitational/debian-tall:stretch
COPY --from=downloader /influx /influxd /influx_inspect /influx_stress /influx_tsm /influxdb/

# Ensure nsswitch is set so localhost will be resolved locally
# https://github.com/gravitational/gravity/issues/1046
# https://github.com/golang/go/issues/35305
RUN set -ex && test -e /etc/nsswitch.conf || echo 'hosts: files dns' > /etc/nsswitch.conf

RUN set -ex && mkdir -p $INFLUXDB_HOME /data /logs

# volume used for storing database logs and data
VOLUME ["/data"]

EXPOSE 8083 8086 8086/udp 8088

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["/usr/bin/telegraf", "--config", "/etc/telegraf/telegraf.conf"]