# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

# Stage 1: Rebuild Go binaries with Go 1.26+ to fix 15 Go stdlib CVEs in
# ini-file, render-template, scuttle, wait-for-port, and yq (were compiled with Go v1.25.0)
FROM golang:1.26-bookworm AS go-builder
RUN go install github.com/bitnami/ini-file@latest && \
    go install github.com/bitnami/render-template@latest && \
    go install github.com/superorbital/scuttle@latest && \
    go install github.com/bitnami/wait-for-port@latest && \
    go install github.com/mikefarah/yq/v4@latest

# Stage 2: Patched os-shell image
FROM docker.io/bitnamilegacy/os-shell:12-debian-12-r51

USER root

# Replace Go binaries compiled with Go 1.25.0 with Go 1.26+ builds.
# Fixes 15 CVEs across all five binaries including CVE-2025-68121 (CRITICAL)
# and 14 HIGH severity stdlib vulnerabilities.
COPY --from=go-builder /go/bin/ini-file         /opt/bitnami/common/bin/ini-file
COPY --from=go-builder /go/bin/render-template  /opt/bitnami/common/bin/render-template
COPY --from=go-builder /go/bin/scuttle          /opt/bitnami/common/bin/scuttle
COPY --from=go-builder /go/bin/wait-for-port    /opt/bitnami/common/bin/wait-for-port
COPY --from=go-builder /go/bin/yq               /opt/bitnami/common/bin/yq

# Remove stale SPDX manifests — they still reference Go 1.25.0 and cause
# false-positive CVE hits in scanners after the binary replacement above.
RUN rm -f /opt/bitnami/common/.spdx-ini-file.spdx \
          /opt/bitnami/common/.spdx-render-template.spdx \
          /opt/bitnami/common/.spdx-scuttle.spdx \
          /opt/bitnami/common/.spdx-wait-for-port.spdx \
          /opt/bitnami/common/.spdx-yq.spdx

# Upgrade all Debian 12 packages to pull in security fixes.
# Fixes: gpgv (CVE-2025-68973), jq/libjq1 (CVE-2025-48060), libcap2 (CVE-2026-4878),
#        libgnutls30 (CVE-2026-33845 CRITICAL, CVE-2026-42010 CRITICAL + 3 HIGH),
#        libnghttp2-14 (CVE-2026-27135), libpam-* (CVE-2025-6020),
#        libssl3/openssl (CVE-2026-31789 CRITICAL + 7 more),
#        libperl5.36/perl/perl-base/perl-modules-5.36 (CVE-2023-31484)
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Remove perl packages — os-shell is a container utility image with no Perl scripts.
# Eliminates CVE-2026-42496 (CRITICAL), CVE-2026-8376 (CRITICAL), CVE-2026-42497,
# CVE-2026-48962, and CVE-2026-9538 which have no available Debian 12 fix.
RUN dpkg --purge --force-depends --force-remove-essential \
        perl \
        perl-modules-5.36 \
        libperl5.36 \
        perl-base

USER 1001
