FROM golang:1.15 as builder
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
RUN useradd --no-create-home --system beats && mkdir /beats && chown beats:beats /beats
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/filebeat/filebeat /beats/filebeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/filebeat/modules.d/ /beats/modules.d/
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/filebeat/build/kibana/ /beats/kibana/
WORKDIR /beats
ENTRYPOINT ["/beats/filebeat", "-e"]
USER beats
