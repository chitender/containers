# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

FROM docker.io/bitnamilegacy/jmx-exporter:0.20.0-debian-12-r11

USER root

# Upgrade all packages to pull in Debian security fixes
# Fixes: gpgv, libc6/libc-bin, libcap2, libfile-find-rule-perl, libgnutls30,
#        liblzma5, libpam-*, libperl5.36/perl (CVE-2023-31484, CVE-2024-56406),
#        libsystemd0, libudev1
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Remove packages not needed at JMX exporter runtime (Java app, no Perl needed).
# --force-remove-essential required for perl-base (Essential: yes in Debian).
RUN dpkg --purge --force-depends --force-remove-essential \
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
