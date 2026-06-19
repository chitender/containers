# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

# Stage 1: Rebuild Go binaries with Go 1.26+ to fix Go stdlib CVEs
# (wait-for-port and render-template were compiled with Go 1.21.x in 3.6)
FROM golang:1.26-bookworm AS go-builder
RUN go install github.com/bitnami/wait-for-port@latest && \
    go install github.com/bitnami/render-template@latest

# Stage 2: Patched Kafka 3.9 image
FROM docker.io/bitnamilegacy/kafka:3.9.0-debian-12-r13

USER root

# Replace Go binaries with Go 1.26+ builds
COPY --from=go-builder /go/bin/wait-for-port   /opt/bitnami/common/bin/wait-for-port
COPY --from=go-builder /go/bin/render-template /opt/bitnami/common/bin/render-template

# Upgrade all packages to pull in Debian 12 security fixes
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Remove perl-base — Kafka is JVM-based, no Perl needed at runtime
RUN dpkg --purge --force-depends --force-remove-essential \
        perl-base

# Delete apt binaries and state — no package manager needed at runtime
RUN rm -rf \
        /usr/bin/apt /usr/bin/apt-get /usr/bin/apt-cache \
        /usr/bin/apt-config /usr/bin/apt-key /usr/bin/apt-mark \
        /usr/lib/apt /etc/apt \
        /var/lib/apt /var/cache/apt \
        /usr/sbin/install_packages /usr/sbin/uninstall_packages

USER 1001
