# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

# Stage 1: Rebuild Go binaries with Go 1.26+ to fix Go stdlib CVEs
FROM golang:1.26-bookworm AS go-builder
RUN go install github.com/bitnami/wait-for-port@latest && \
    go install github.com/bitnami/render-template@latest

# Stage 2: Download patched Java JARs from Maven Central
FROM maven:3-eclipse-temurin-17 AS jar-downloader
WORKDIR /jars
RUN mvn --batch-mode dependency:copy -Dartifact=commons-beanutils:commons-beanutils:1.11.0          -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=io.netty:netty-codec:4.1.135.Final                  -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=io.netty:netty-handler:4.1.135.Final                -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=org.bitbucket.b_c:jose4j:0.9.6                     -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=org.codehaus.plexus:plexus-utils:3.6.1              -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=org.eclipse.jetty:jetty-http:9.4.57.v20241219       -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=org.eclipse.jetty:jetty-server:9.4.57.v20241219     -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=org.lz4:lz4-java:1.8.1                             -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=org.apache.zookeeper:zookeeper:3.8.6               -DoutputDirectory=/jars && \
    mvn --batch-mode dependency:copy -Dartifact=org.apache.zookeeper:zookeeper-jute:3.8.6          -DoutputDirectory=/jars

# Stage 3: Patched Kafka 3.9 image
FROM docker.io/bitnamilegacy/kafka:3.9.0-debian-12-r13

USER root

# Replace Go binaries with Go 1.26+ builds
COPY --from=go-builder /go/bin/wait-for-port   /opt/bitnami/common/bin/wait-for-port
COPY --from=go-builder /go/bin/render-template /opt/bitnami/common/bin/render-template

# Replace vulnerable Java JARs with patched versions
# Old JARs removed by wildcard; new JARs copied from jar-downloader stage
RUN LIBS=/opt/bitnami/kafka/libs && \
    rm -f ${LIBS}/commons-beanutils-*.jar      && \
    rm -f ${LIBS}/netty-codec-*.jar            && \
    rm -f ${LIBS}/netty-handler-*.jar          && \
    rm -f ${LIBS}/jose4j-*.jar                 && \
    rm -f ${LIBS}/plexus-utils-*.jar           && \
    rm -f ${LIBS}/jetty-http-*.jar             && \
    rm -f ${LIBS}/jetty-server-*.jar           && \
    rm -f ${LIBS}/lz4-java-*.jar               && \
    rm -f ${LIBS}/zookeeper-[0-9]*.jar         && \
    rm -f ${LIBS}/zookeeper-jute-*.jar
COPY --from=jar-downloader /jars/ /opt/bitnami/kafka/libs/

# Upgrade all packages to pull in Debian 12 security fixes
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Remove Perl packages — Kafka is JVM-based, no Perl needed at runtime
RUN dpkg --purge --force-depends --force-remove-essential \
        perl-base && \
    dpkg --purge --force-depends \
        perl perl-modules-5.36 libperl5.36

# Delete apt binaries and state — no package manager needed at runtime
RUN rm -rf \
        /usr/bin/apt /usr/bin/apt-get /usr/bin/apt-cache \
        /usr/bin/apt-config /usr/bin/apt-key /usr/bin/apt-mark \
        /usr/lib/apt /etc/apt \
        /var/lib/apt /var/cache/apt \
        /usr/sbin/install_packages /usr/sbin/uninstall_packages

USER 1001
