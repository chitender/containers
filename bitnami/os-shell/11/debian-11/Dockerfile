# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

# Stage 1: Rebuild Go binaries with Go 1.26+ to fix 18 Go stdlib CVEs in
# ini-file, render-template, wait-for-port, and yq (were compiled with Go v1.21.4)
FROM golang:1.26-bookworm AS go-builder
RUN go install github.com/bitnami/ini-file@latest && \
    go install github.com/bitnami/render-template@latest && \
    go install github.com/bitnami/wait-for-port@latest && \
    go install github.com/mikefarah/yq/v4@latest

# Stage 2: Patched os-shell image
FROM docker.io/bitnamilegacy/os-shell:11-debian-11-r92

USER root

# Replace Go binaries compiled with Go 1.21.4 with Go 1.26+ builds.
# Fixes 18 CVEs across all four binaries including CVE-2024-24790 (CRITICAL),
# CVE-2025-68121 (CRITICAL), and 16 HIGH severity stdlib vulnerabilities.
COPY --from=go-builder /go/bin/ini-file         /opt/bitnami/common/bin/ini-file
COPY --from=go-builder /go/bin/render-template  /opt/bitnami/common/bin/render-template
COPY --from=go-builder /go/bin/wait-for-port    /opt/bitnami/common/bin/wait-for-port
COPY --from=go-builder /go/bin/yq               /opt/bitnami/common/bin/yq

# Upgrade all Debian 11 packages to pull in security fixes.
# Fixes: gpgv (CVE-2025-68973), jq/libjq1 (CVE-2025-48060, CVE-2026-32316, CVE-2026-40164),
#        libc-bin/libc6 (CVE-2024-2961, CVE-2024-33599), libcom-err2 (CVE-2022-1304),
#        libgnutls30 (CVE-2026-33845 CRITICAL + 8 more), libgssapi-krb5-2/libk5crypto3/
#        libkrb5-3/libkrb5support0 (CVE-2024-37371 CRITICAL, CVE-2024-37370),
#        libnghttp2-14 (CVE-2026-27135), libpam-* (CVE-2025-6020),
#        libssh2-1 (CVE-2020-22218), libssl1.1/openssl (CVE-2025-69421 + 5 more),
#        libsystemd0/libudev1 (CVE-2023-50387, CVE-2023-50868),
#        perl-base (CVE-2020-16156, CVE-2023-31484, CVE-2023-47038)
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Remove perl-base — os-shell is a container utility image with no Perl scripts.
# Eliminates CVE-2026-42496 (CRITICAL), CVE-2026-8376 (CRITICAL), CVE-2026-42497,
# CVE-2026-48962, and CVE-2026-9538 which have no available Debian 11 fix.
RUN dpkg --purge --force-depends --force-remove-essential perl-base

USER 1001
