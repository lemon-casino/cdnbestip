FROM alpine:latest AS cfst-builder
WORKDIR /app
ARG TARGETOS=linux
ARG TARGETARCH=amd64
ARG TARGETVARIANT
RUN apk add --no-cache curl jq
RUN set -eux; \
    case "${TARGETARCH}/${TARGETVARIANT}" in \
        amd64/*) asset_arch="amd64" ;; \
        386/*) asset_arch="386" ;; \
        arm64/*) asset_arch="arm64" ;; \
        arm/v5) asset_arch="armv5" ;; \
        arm/v6) asset_arch="armv6" ;; \
        arm/v7|arm/) asset_arch="armv7" ;; \
        *) echo "Unsupported CloudflareSpeedTest platform: ${TARGETOS}/${TARGETARCH}/${TARGETVARIANT}" >&2; exit 1 ;; \
    esac; \
    test "${TARGETOS}" = "linux"; \
    release_asset="cfst_${TARGETOS}_${asset_arch}.tar.gz"; \
    download_url="$(curl -fsSL -H 'Accept: application/vnd.github+json' -H 'User-Agent: cdnbestip-docker-build' \
        'https://api.github.com/repos/XIU2/CloudflareSpeedTest/releases/latest' \
        | jq -er --arg asset "${release_asset}" '.assets[] | select(.name == $asset) | .browser_download_url')"; \
    curl -fsSL "${download_url}" | tar -xzvf -; \
    cp "/app/cfst_linux_${asset_arch}/cfst" /app/cfst; \
    chmod +x /app/cfst; \
    test -x /app/cfst

#####################################################################################
# Builder
FROM python:3.13-alpine AS builder
WORKDIR /build
COPY . .
RUN pip install --upgrade pip build
RUN python -m build --wheel

#####################################################################################
FROM python:3.13-alpine
LABEL maintainer="Jetsung Chan<i@jetsung.com>"

WORKDIR /app

RUN apk add --no-cache cronie

COPY --from=cfst-builder /app/cfst /usr/local/bin/cfst
COPY --from=builder /build/dist /app
COPY README.md /app/README.md

RUN mkdir ~/.cache

RUN pip install /app/*.whl && \
    rm -rf /app/*.whl && \
    rm -rf /var/cache/apk/*

RUN printf "%b" '#!'"/usr/bin/env sh\n \
if [ \"\$1\" = \"daemon\" ];  then \n \
 exec crond -n -s -m off \n \
else \n \
 exec -- \"\$@\"\n \
fi\n" >/entry.sh && chmod +x /entry.sh

VOLUME /app

ENTRYPOINT ["/entry.sh"]
