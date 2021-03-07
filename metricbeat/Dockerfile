FROM golang:latest as builder
ARG BEATSVERSION=master
RUN apt-get -y update && apt-get -y --no-install-recommends install python3
RUN go get -u -d github.com/magefile/mage && \
    cd $GOPATH/src/github.com/magefile/mage && \
    go run bootstrap.go
RUN mkdir -p ${GOPATH}/src/github.com/elastic && \
    git clone --branch ${BEATSVERSION} https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats
RUN cd ${GOPATH}/src/github.com/elastic/beats/metricbeat/ && \
    mage build && \
    mage dashboards

FROM centos:8
RUN useradd --no-create-home --system beats && mkdir /usr/share/metricbeat && chown beats:beats /usr/share/metricbeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/metricbeat/metricbeat /usr/share/metricbeat/metricbeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/metricbeat/modules.d/ /usr/share/metricbeat/modules.d/
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/metricbeat/build/kibana/ /usr/share/metricbeat/kibana/
WORKDIR /usr/share/metricbeat
CMD ["/usr/share/metricbeat/metricbeat", "-e"]
USER beats
