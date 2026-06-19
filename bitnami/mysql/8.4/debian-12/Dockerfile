# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

FROM docker.io/bitnamilegacy/mysql:8.4.5-debian-12-r0

USER root

RUN apt-get update && apt-get upgrade -y && \
    apt-get purge -y --auto-remove \
        gpgv \
        libfile-find-rule-perl \
        openssl \
        libpam-modules libpam-modules-bin libpam-runtime \
        perl perl-modules-5.36 && \
    apt-get purge -y --auto-remove apt apt-utils && \
    rm -rf /var/lib/apt /var/cache/apt /usr/lib/apt /etc/apt && \
    rm -f /usr/sbin/install_packages /usr/sbin/uninstall_packages

USER 1001
