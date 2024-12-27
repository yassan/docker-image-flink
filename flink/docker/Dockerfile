# syntax=docker/dockerfile:1
ARG base_image=unknown

FROM alpine:3 AS download

ARG flink_version=unknown

WORKDIR /download

RUN apk add -U curl gnupg tar

# Download deps jars
ENV FLINK_URL=https://dlcdn.apache.org/flink/flink-${flink_version}/flink-${flink_version}-bin-scala_2.12.tgz

RUN <<EOF
mkdir -p /download/flink
curl -f -sL --retry 3 "${FLINK_URL}" | tar xz --strip-components 1 -C /download/flink
EOF


#####################################################################
FROM ${base_image}

RUN mkdir -p ${FLINK_HOME}/plugins/s3

ARG flink_version=unknown

COPY --from=download --chown=flink:flink --chmod=755 /download/flink/opt/flink-s3-fs-hadoop-${flink_version}.jar  ${FLINK_HOME}/plugins/s3/
