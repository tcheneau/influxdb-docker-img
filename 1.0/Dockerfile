FROM container4armhf/armhf-alpine
MAINTAINER Tony Cheneau <tony.cheneau@amnesiak.org>

RUN echo "http://nl.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
RUN apk update \
	&& apk add --no-cache git go bash musl-dev

ENV INFLUXDB_VERSION v1.2.3
ENV GOPATH /tmp/go

RUN go get github.com/sparrc/gdm
ENV PATH=$PATH:$GOPATH/bin

# Build InfluxDB from source
RUN mkdir -p $GOPATH/src/github.com/influxdata \
	&& cd $GOPATH/src/github.com/influxdata \
	&& git clone --depth 1 https://github.com/influxdata/influxdb --branch ${INFLUXDB_VERSION}\
	&& cd influxdb \
	&& gdm restore \
	#&& git checkout tags/v${INFLUXDB_VERSION} \
	# go get -u sometime fails first time, but succeeds second time
	# (see https://github.com/golang/go/issues/9224)
	# therefore, we try it a second time, if it failed the first time
	# && ( go get -u -f ./... || go get -u -f ./... || true) \
	&& go clean ./... \
	&& go install ./... \
	&& ls $GOPATH/bin \
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
