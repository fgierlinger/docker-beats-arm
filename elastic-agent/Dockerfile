FROM golang:1.15 as builder
ARG BEATSVERSION=master
RUN apt-get -y update && apt-get -y --no-install-recommends install python3
RUN go get -u -d github.com/magefile/mage && \
    cd $GOPATH/src/github.com/magefile/mage && \
    go run bootstrap.go
RUN mkdir -p ${GOPATH}/src/github.com/elastic && \
    git clone --branch ${BEATSVERSION} https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats
RUN cd ${GOPATH}/src/github.com/elastic/beats/x-pack/elastic-agent && \
    mage build:binary

FROM centos:8
RUN useradd --no-create-home --system beats && mkdir /usr/share/elastic_agent && chown beats:beats /usr/share/elastic_agent
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/x-pack/elastic-agent/build/elastic-agent /usr/share/elastic_agent/elastic-agent
COPY --from=builder --chown=beats:beats /go/src/github.com/elastic/beats/x-pack/elastic-agent/elastic-agent.docker.yml /usr/share/elastic_agent/elastic-agent.yml
WORKDIR /beats
ENTRYPOINT ["/usr/share/elastic_agent/elastic-agent", "-e"]
USER beats
