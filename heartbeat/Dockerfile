FROM golang:1.15 as builder
ARG BEATSVERSION=master
RUN apt-get -y update && apt-get -y --no-install-recommends install python3
RUN go get -u -d github.com/magefile/mage && \
    cd $GOPATH/src/github.com/magefile/mage && \
    go run bootstrap.go
RUN mkdir -p ${GOPATH}/src/github.com/elastic && \
    git clone --branch ${BEATSVERSION} https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats
RUN cd ${GOPATH}/src/github.com/elastic/beats/heartbeat/ && \
    mage build

FROM centos:8
RUN useradd --no-create-home --system beats && mkdir /usr/share/heartbeat/ && chown beats:beats /usr/share/heartbeat/
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/heartbeat/heartbeat /usr/share/heartbeat/heartbeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/heartbeat/monitors.d/ /usr/share/heartbeat/monitors.d/
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/heartbeat/monitors/ /usr/share/heartbeat/monitors/
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/heartbeat/heartbeat.docker.yml /usr/share/heartbeat/heartbeat.yml
WORKDIR /usr/share/heartbeat/
ENTRYPOINT ["/usr/share/heartbeat/heartbeat", "-e"]
USER beats
