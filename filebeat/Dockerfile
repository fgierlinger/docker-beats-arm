FROM golang:latest as builder
ARG BEATSVERSION=master
RUN apt-get -y update && apt-get -y --no-install-recommends install python3
RUN go get -u -d github.com/magefile/mage && \
    cd $GOPATH/src/github.com/magefile/mage && \
    go run bootstrap.go
RUN mkdir -p ${GOPATH}/src/github.com/elastic && \
    git clone --branch ${BEATSVERSION} https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats
RUN cd ${GOPATH}/src/github.com/elastic/beats/filebeat/ && \
    mage build && \
    mage dashboards

FROM centos:8
RUN useradd --no-create-home --system beats && mkdir /usr/share/filebeat && chown beats:beats /usr/share/filebeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/filebeat/filebeat /usr/share/filebeat/filebeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/filebeat/modules.d/ /usr/share/filebeat/modules.d/
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/filebeat/build/kibana/ /usr/share/filebeat/kibana/
WORKDIR /usr/share/filebeat
CMD ["/usr/share/filebeat/filebeat", "-e"]
USER beats
