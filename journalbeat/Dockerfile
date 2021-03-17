FROM golang:1.15 as builder
ARG BEATSVERSION=master
RUN apt-get -y update && apt-get -y --no-install-recommends install python3 libsystemd-dev
RUN go get -u -d github.com/magefile/mage && \
    cd $GOPATH/src/github.com/magefile/mage && \
    go run bootstrap.go
RUN mkdir -p ${GOPATH}/src/github.com/elastic && \
    git clone --branch ${BEATSVERSION} https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats
RUN cd ${GOPATH}/src/github.com/elastic/beats/journalbeat/ && \
    mage build

FROM centos:8
RUN useradd --no-create-home --system beats && mkdir /usr/share/journalbeat && chown beats:beats /usr/share/journalbeat
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/journalbeat/journalbeat /usr/share/journalbeat/journalbeat
WORKDIR /usr/share/journalbeat/
CMD ["/usr/share/journalbeat/journalbeat", "-e"]
USER beats
