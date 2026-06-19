# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

# Stage 1: Rebuild ini-file with Go 1.26+ to fix 15 Go stdlib CVEs
FROM golang:1.26-bookworm AS ini-file-builder
RUN go install github.com/bitnami/ini-file@latest

# Stage 2: Patched MySQL image
FROM docker.io/bitnamilegacy/mysql:8.4.5-debian-12-r0

USER root

# Replace the ini-file binary (was compiled with Go 1.23.8) with one
# built against Go 1.26+ stdlib — fixes all 15 CVEs in the gobinary scan
COPY --from=ini-file-builder /go/bin/ini-file /opt/bitnami/common/bin/ini-file

# Upgrade all packages to pull in Debian security fixes
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Remove packages not needed at MySQL runtime.
# perl/libperl5.36/perl-base: MySQL is a compiled binary, no script uses Perl.
# openssl CLI: MySQL links libssl3 (the library), not the CLI tool.
# --force-depends bypasses reverse-dependency checks safe in a container.
RUN dpkg --purge --force-depends \
        openssl \
        libfile-find-rule-perl \
        perl \
        perl-modules-5.36 \
        libperl5.36 \
        perl-base

# Delete apt binaries and state — no package manager needed at runtime
RUN rm -rf \
        /usr/bin/apt /usr/bin/apt-get /usr/bin/apt-cache \
        /usr/bin/apt-config /usr/bin/apt-key /usr/bin/apt-mark \
        /usr/lib/apt /etc/apt \
        /var/lib/apt /var/cache/apt \
        /usr/sbin/install_packages /usr/sbin/uninstall_packages

USER 1001
