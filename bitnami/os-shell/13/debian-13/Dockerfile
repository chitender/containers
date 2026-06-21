# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

# Stage 1: Build all Go utilities with Go 1.26+ stdlib
# Produces clean binaries with no Go stdlib CVEs
FROM golang:1.26-bookworm AS go-builder
# Clear credential helper so go's internal git can clone public repos
# without prompting (golang base image ships one that requires a TTY)
RUN git config --global credential.helper ""
RUN go install github.com/bitnami/ini-file@latest && \
    go install github.com/bitnami/render-template@latest && \
    go install github.com/bitnami/wait-for-port@latest && \
    go install github.com/mikefarah/yq/v4@latest
# scuttle is not on the Go module proxy; use tags API (repo has no releases)
# to resolve the latest tag, then download and build from the tarball
RUN SCUTTLE_TAG=$(curl -fsSL "https://api.github.com/repos/superorbital/scuttle/tags" \
        | grep '"name"' | head -1 | sed 's/.*"name": "\([^"]*\)".*/\1/') \
    && if [ -z "${SCUTTLE_TAG}" ]; then echo "ERROR: could not resolve scuttle tag" && exit 1; fi \
    && echo "Building scuttle ${SCUTTLE_TAG}" \
    && mkdir -p /tmp/scuttle \
    && curl -fsSL "https://github.com/superorbital/scuttle/archive/refs/tags/${SCUTTLE_TAG}.tar.gz" \
       | tar -xz -C /tmp/scuttle --strip-components=1 \
    && cd /tmp/scuttle \
    && CGO_ENABLED=0 go build -ldflags="-s -w" -o /go/bin/scuttle . \
    && rm -rf /tmp/scuttle

# Stage 2: Debian 13 Trixie base — clean slate, no legacy CVE baggage
FROM debian:trixie-slim

ARG TARGETARCH
ENV HOME="/" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="debian-13" \
    OS_NAME="linux"

# Install only what os-shell needs; apt-get upgrade ensures latest point-release fixes
RUN apt-get update && apt-get install -y --no-install-recommends \
        bash \
        ca-certificates \
        curl \
        jq \
        procps \
    && apt-get upgrade -y \
    && apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

RUN mkdir -p /opt/bitnami/common/bin

COPY --from=go-builder /go/bin/ini-file         /opt/bitnami/common/bin/ini-file
COPY --from=go-builder /go/bin/render-template  /opt/bitnami/common/bin/render-template
COPY --from=go-builder /go/bin/scuttle          /opt/bitnami/common/bin/scuttle
COPY --from=go-builder /go/bin/wait-for-port    /opt/bitnami/common/bin/wait-for-port
COPY --from=go-builder /go/bin/yq               /opt/bitnami/common/bin/yq

RUN useradd -r -u 1001 -g root os-shell
RUN find / -perm /6000 -type f -exec chmod a-s {} \; || true

ENV APP_VERSION="13" \
    BITNAMI_APP_NAME="os-shell" \
    PATH="/opt/bitnami/common/bin:$PATH"

USER 1001
