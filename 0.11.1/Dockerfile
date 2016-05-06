FROM tutum/curl:trusty
MAINTAINER Catalin Jora <catalin.jora@container-solutions.com>

# Install InfluxDB
ENV INFLUXDB_VERSION 0.11.1-1
RUN curl -s -o /tmp/influxdb_latest_amd64.deb https://s3.amazonaws.com/influxdb/influxdb_${INFLUXDB_VERSION}_amd64.deb && \
  dpkg -i /tmp/influxdb_latest_amd64.deb && \
  rm /tmp/influxdb_latest_amd64.deb && \
  rm -rf /var/lib/apt/lists/*

ADD types.db /usr/share/collectd/types.db
ADD config.toml /config/config.toml
ADD run.sh /run.sh
RUN chmod +x /*.sh

#ENV PRE_CREATE_DB mon
ENV SSL_SUPPORT **False**
ENV SSL_CERT **None**

# Admin server WebUI
EXPOSE 8083

# HTTP API
EXPOSE 8086

# Cluster meta communication
EXPOSE 8091

# Cluster communication
EXPOSE 8088

#VOLUME ["/var/lib/influxdb"]

CMD ["/run.sh"]
