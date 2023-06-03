ARG VERSION="main"
ARG REVISION="f3a07934b49bf142925a5913e3e19f3528eda0d2"

FROM golang:alpine AS builder

ARG VERSION
ARG REVISION

ENV GOOS=linux
ENV GO111MODULE=on

RUN apk upgrade && apk --no-cache add \
	build-base \
    btrfs-progs-dev \
    btrfs-progs-libs \
    btrfs-progs-static \
    ca-certificates \
    git \
    gcc \
    make 

RUN mkdir -p /go/src/ \
    && git clone --branch "${VERSION}" https://github.com/containerd/containerd.git /go/src/containerd \
	&& cd /go/src/containerd \
    && git reset --hard "${REVISION}" \
    && make EXTRA_FLAGS="-buildmode pie" \
	EXTRA_LDFLAGS='-linkmode external -extldflags "-fno-PIC -static"' \
	BUILDTAGS="netgo osusergo static_build" \
	&& cp -vr ./bin/* /usr/local/bin/ \
	&& if [$TARGETPLATFORM = "linux/amd64"]; then apk add --no-cache upx && find /usr/local/bin/ -executable -type f -exec upx --lzma --best {} \;; fi 

RUN addgroup containerd \
    && adduser -G containerd -s /bin/sh -D containerd

FROM scratch

ARG VERSION
ARG REVISION

ENV CONTAINERD_VERSION="${VERSION}" \
    CONTAINERD_REVISION="${REVISION}"

COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /usr/local/bin/ /usr/local/bin/

USER containerd

ENTRYPOINT ["/usr/local/bin/containerd"]
CMD ["--help"]

LABEL org.opencontainers.image.title="containerd" \
    org.opencontainers.image.description="containerd in Docker" \ 
    org.opencontainers.image.revision="${REVISION}" \
    org.opencontainers.image.version="${VERSION}"

