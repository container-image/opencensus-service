ARG GOLANG_VERSION
ARG ALPINE_VERSION
FROM docker.io/golang:${GOLANG_VERSION}-alpine${ALPINE_VERSION} AS ocagent-builder
ENV OUTDIR=/out
RUN set -eux && \
	apk add --no-cache \
		bzr \
		ca-certificates \
		git
RUN set -eux && \
	go get -d -v github.com/census-instrumentation/opencensus-service || true
RUN set -eux && \
	mkdir -p "${OUTDIR}/usr/bin" && \
	cd /go/src/github.com/census-instrumentation/opencensus-service && \
	GO111MODULE=on go mod tidy -v && \
	GO111MODULE=on go mod vendor -v && \
	GO111MODULE=off GOBIN=${OUTDIR}/usr/bin/ go install -a -v -tags 'osusergo netgo static static_build' -installsuffix netgo -ldflags='-d -s -w -extldflags=-static' \
		github.com/census-instrumentation/opencensus-service/cmd/ocagent

FROM scratch AS ocagent
COPY --from=ocagent-builder /out/ /
COPY --from=ocagent-builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
EXPOSE 55678 55679
ENTRYPOINT ["/usr/bin/ocagent"]
