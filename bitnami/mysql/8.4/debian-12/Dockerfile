# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

FROM docker.io/bitnamilegacy/mysql:8.4.5-debian-12-r0

USER root

# Upgrade all packages to pull in Debian security fixes
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Remove packages not needed at MySQL runtime.
# --force-depends bypasses reverse-dependency checks safe in a container.
RUN dpkg --purge --force-depends \
        openssl \
        libfile-find-rule-perl \
        perl \
        perl-modules-5.36

# Delete apt binaries and state — no package manager needed at runtime
RUN rm -rf \
        /usr/bin/apt /usr/bin/apt-get /usr/bin/apt-cache \
        /usr/bin/apt-config /usr/bin/apt-key /usr/bin/apt-mark \
        /usr/lib/apt /etc/apt \
        /var/lib/apt /var/cache/apt \
        /usr/sbin/install_packages /usr/sbin/uninstall_packages

USER 1001
