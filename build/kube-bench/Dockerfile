FROM golang:1.14.6 AS build

ARG TARGETPLATFORM
ARG VERSION

ENV GO111MODULE=on \
    CGO_ENABLED=0

WORKDIR /go/src/github.com/aquasecurity/kube-bench/

RUN export GOOS=$(echo ${TARGETPLATFORM} | cut -d / -f1) && \
    export GOARCH=$(echo ${TARGETPLATFORM} | cut -d / -f2) && \
    GOARM=$(echo ${TARGETPLATFORM} | cut -d / -f3 | cut -c2-) && \
    git clone https://github.com/aquasecurity/kube-bench.git . && \
    go install -a -ldflags "-X github.com/aquasecurity/kube-bench/cmd.KubeBenchVersion=${VERSION} -w"

FROM alpine:3.12 AS run

ARG BUILD_DATE
ARG VCS_REF

ENV PATH=$PATH:/usr/local/mount-from-host/bin

WORKDIR /opt/kube-bench/

LABEL org.label-schema.build-date=$BUILD_DATE \
    org.label-schema.name="kube-bench" \
    org.label-schema.description="Run the CIS Kubernetes Benchmark tests" \
    org.label-schema.url="https://github.com/aquasecurity/kube-bench" \
    org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.vcs-url="https://github.com/aquasecurity/kube-bench" \
    org.label-schema.schema-version="1.0"

# add GNU ps for -C, -o cmd, and --no-headers support
# https://github.com/aquasecurity/kube-bench/issues/109
RUN apk --no-cache add procps

# Openssl is used by OpenShift tests
# https://github.com/aquasecurity/kube-bench/issues/535
RUN apk --no-cache add openssl

COPY --from=build /go/bin/kube-bench /usr/local/bin/kube-bench

COPY --from=build /go/src/github.com/aquasecurity/kube-bench/entrypoint.sh .

COPY --from=build /go/src/github.com/aquasecurity/kube-bench/cfg/ cfg/

ENTRYPOINT ["./entrypoint.sh"]

CMD ["install"]
