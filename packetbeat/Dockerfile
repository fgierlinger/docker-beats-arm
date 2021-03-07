FROM golang:latest as builder
ARG BEATSVERSION=master
RUN apt-get -y update && apt-get -y --no-install-recommends install python3 libpcap-dev
RUN go get -u -d github.com/magefile/mage && \
    cd $GOPATH/src/github.com/magefile/mage && \
    go run bootstrap.go
RUN mkdir -p ${GOPATH}/src/github.com/elastic && \
    git clone --branch ${BEATSVERSION} https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats
RUN cd ${GOPATH}/src/github.com/elastic/beats/packetbeat/ && \
    mage build && \
    mage dashboards

FROM debian:10
RUN apt-get -y update && apt-get -y --no-install-recommends install libpcap-dev
RUN useradd --no-create-home --system beats && mkdir /usr/share/packetbeat && chown beats:beats /usr/share/packetbeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/packetbeat/packetbeat /usr/share/packetbeat/packetbeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/packetbeat/build/kibana/ /usr/share/packetbeat/kibana/
WORKDIR /usr/share/packetbeat
CMD ["/usr/share/packetbeat/packetbeat", "-e"]
USER beats
