FROM golang:1.15 as builder
ARG BEATSVERSION=master
RUN apt-get -y update && apt-get -y --no-install-recommends install python3
RUN go get -u -d github.com/magefile/mage && \
    cd $GOPATH/src/github.com/magefile/mage && \
    go run bootstrap.go
RUN mkdir -p ${GOPATH}/src/github.com/elastic && \
    git clone --branch ${BEATSVERSION} https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats
RUN cd ${GOPATH}/src/github.com/elastic/beats/x-pack/auditbeat/ && \
    mage build && \
    mage dashboards

FROM centos:8
RUN useradd --no-create-home --system beats && mkdir /usr/share/auditbeat && chown beats:beats /usr/share/auditbeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/x-pack/auditbeat/auditbeat /usr/share/auditbeat/auditbeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/x-pack/auditbeat/module /usr/share/auditbeat/module
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/x-pack/auditbeat/build/kibana/ /usr/share/auditbeat/kibana/
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/x-pack/auditbeat/auditbeat.docker.yml /usr/share/auditbeat/auditbeat.yml
RUN dnf install -y -q audit
WORKDIR /usr/share/auditbeat
CMD ["/usr/share/auditbeat/auditbeat", "-e"]
USER beats
