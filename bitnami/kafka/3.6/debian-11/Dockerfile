# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

# Stage 1: Rebuild Go binaries with Go 1.26+ to fix 18 Go stdlib CVEs in
# render-template and wait-for-port (were compiled with Go v1.21.4)
FROM golang:1.26-bookworm AS go-builder
RUN go install github.com/bitnami/render-template@latest && \
    go install github.com/bitnami/wait-for-port@latest

# Stage 2: Patched Kafka image
FROM docker.io/bitnamilegacy/kafka:3.6.0-debian-11-r2

USER root

# Replace Go binaries compiled with Go 1.21.4 with Go 1.26+ builds
COPY --from=go-builder /go/bin/render-template /opt/bitnami/common/bin/render-template
COPY --from=go-builder /go/bin/wait-for-port   /opt/bitnami/common/bin/wait-for-port

# Upgrade all Debian 11 packages to pull in security fixes.
# Fixes: gpgv, libc-bin/libc6, libcom-err2, libgnutls30, libgssapi-krb5-2,
#        libk5crypto3, libkrb5-3, libkrb5support0, libnghttp2-14, libpam-*,
#        libssh2-1, libssl1.1, libsystemd0, libudev1, openssl, perl-base
#        (CVE-2020-16156, CVE-2023-31484, CVE-2023-47038)
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Remove packages not needed at Kafka runtime (JVM-based app, no Perl needed).
# curl/libcurl4 kept — Kafka entrypoint scripts use curl for broker readiness checks.
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
