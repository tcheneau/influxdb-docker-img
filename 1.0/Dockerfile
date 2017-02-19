FROM armhf-archlinux
MAINTAINER Tony Cheneau <tony.cheneau@amnesiak.org>

RUN pacman -Sy \
	&& pacman --noconfirm -S git go

ENV INFLUXDB_VERSION 1.1.1
ENV GOPATH /tmp/go

# Build InfluxDB from source
RUN mkdir -p $GOPATH/src/github.com/influxdata \
	&& cd $GOPATH/src/github.com/influxdata \
	&& git clone https://github.com/influxdata/influxdb \
	&& cd influxdb \
	#&& git checkout tags/v${INFLUXDB_VERSION} \
	&& git checkout master \
	# go get -u sometime fails first time, but succeeds second time
	# (see https://github.com/golang/go/issues/9224)
	# therefore, we try it a second time, if it failed the first time
	&& ( go get -u -f ./... || go get -u -f ./... || true) \
	&& go build ./... \
	&& mv $GOPATH/bin/* /usr/bin \
	&& rm -rf $GOPATH/*

ADD types.db /usr/share/collectd/types.db
ADD config.toml /config/config.toml
ADD run.sh /run.sh
RUN chmod +x /*.sh

ENV PRE_CREATE_DB **None**
ENV SSL_SUPPORT **False**
ENV SSL_CERT **None**

# Admin server WebUI
EXPOSE 8083

# HTTP API
EXPOSE 8086

# Raft port (for clustering, don't expose publicly!)
#EXPOSE 8090

# Protobuf port (for clustering, don't expose publicly!)
#EXPOSE 8099

VOLUME ["/data"]

CMD ["/run.sh"]
