ARG BUILD_FROM=ghcr.io/hassio-addons/base:20.0.1
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set workdir
WORKDIR /opt

# Copy Python requirements file
COPY requirements.txt /opt/

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base
ARG UPTIME_KUMA_VERSION="2.2.0"
ARG CLOUDFLARED_VERSION="2026.2.0"
ARG BUILD_ARCH=amd64
# hadolint ignore=DL3003,DL3042
RUN \
    apk add --no-cache --virtual .build-dependencies \
        build-base=0.5-r3 \
        py3-pip=25.1.1-r1 \
    \
    && apk add --no-cache \
        iputils=20250605-r0 \
        nodejs=24.13.0-r1 \
        npm=11.11.0-r0 \
        python3=3.12.12-r0 \
        setpriv=2.41.2-r0 \
    \
    && mkdir -p /opt/uptime-kuma \
    && curl -L -s "https://github.com/louislam/uptime-kuma/archive/refs/tags/${UPTIME_KUMA_VERSION}.tar.gz" \
        | tar zxvf - --strip-components 1 -C /opt/uptime-kuma \
    \
    && cd /opt/uptime-kuma \
    \
    && pip install -r /opt/requirements.txt \
    \
    && npm ci \
        --no-audit \
        --no-fund \
        --no-update-notifier \
        --omit=dev \
    && npm run download-dist \
    \
    && if [ "${BUILD_ARCH}" = "aarch64" ]; then CLOUDFLARED_ARCH="arm64"; \
    elif [ "${BUILD_ARCH}" = "amd64" ]; then CLOUDFLARED_ARCH="amd64"; fi \
    && curl -L --fail -o /usr/bin/cloudflared \
        "https://github.com/cloudflare/cloudflared/releases/download/${CLOUDFLARED_VERSION}/cloudflared-linux-${CLOUDFLARED_ARCH}" \
    && chmod +x /usr/bin/cloudflared \
    \
    && npm cache clear --force \
    \
    && apk del --no-cache --purge .build-dependencies \
    && rm -fr \
        /tmp/* \
        /root/.cache \ 
        /root/.npm \
        /root/.npmrc

# Copy root filesystem
COPY rootfs /

# Build arguments
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION


# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <opensource@frenck.dev>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Apps" \
    org.opencontainers.image.authors="Franck Nijhof <opensource@frenck.dev>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://frenck.dev/home-assistant-apps" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
