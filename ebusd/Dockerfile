# ARG before any FROM so it can be used in the FROM below (Docker requirement).
# Home Assistant's multi-arch Debian base; CI and Supervisor (local builds) both
# build from this Dockerfile alone, there is no build.yaml anymore.
ARG BUILD_FROM=ghcr.io/home-assistant/base-debian:trixie

# Pull ebusd binary and config directly from john30's official multi-arch image.
# Docker automatically selects the right arch variant from the manifest.
# The tag here is kept in sync with config.yaml (single source of truth) by
# update-ebusd.yml, which bumps both files together in each version-bump PR.
FROM john30/ebusd:v26.1 AS upstream

# ttyd: web terminal (xterm.js over WebSocket) served on the HA ingress port.
# Multi-arch image — Docker picks the right variant automatically. Pinned so that
# builds are reproducible; Dependabot proposes updates.
FROM ghcr.io/tsl0922/ttyd:1.7.8 AS ttyd-upstream

FROM ${BUILD_FROM}

ENV LANG C.UTF-8

# Install only the runtime libraries the ebusd binary links against,
# plus python3/sqlite3 for the optional SQLite telegram log (ebus_sqlite.py)
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      libmosquitto1 \
      ca-certificates \
      netcat-openbsd \
      python3 \
      sqlite3 \
 && rm -rf /var/lib/apt/lists/*

COPY --from=upstream /usr/bin/ebusctl /usr/bin/ebusctl

# ebusd itself comes from the static release of aghulas/ebusd (runtime CRC for Weishaupt),
# checked against the pinned SHA-256 of the binary for the architecture being built.
ARG BUILD_ARCH
ARG EBUSD_RELEASE=v26.1-weishaupt.2
ARG EBUSD_SHA256_aarch64=c7854b8aa5852451522b34ac253986c8ebbd74b23b27258d4d492f36e5aeec1d
ARG EBUSD_SHA256_amd64=587ef8962e89f6a894b8e8c4c4962589ff3e2cc257b09df15d2cb5c01ce165c3
RUN set -e; \
    case "${BUILD_ARCH}" in \
      aarch64) sha="${EBUSD_SHA256_aarch64}" ;; \
      amd64) sha="${EBUSD_SHA256_amd64}" ;; \
      *) echo "unsupported architecture ${BUILD_ARCH}"; exit 1 ;; \
    esac; \
    curl -fsSL -o /usr/bin/ebusd \
      "https://github.com/aghulas/ebusd/releases/download/${EBUSD_RELEASE}/ebusd-${BUILD_ARCH}"; \
    echo "${sha}  /usr/bin/ebusd" | sha256sum -c -; \
    chmod 755 /usr/bin/ebusd; \
    /usr/bin/ebusd --version
COPY --from=upstream /etc/ebusd/      /etc/ebusd/
COPY --from=ttyd-upstream /usr/bin/ttyd /usr/bin/ttyd

LABEL \
    Description="ebusd" \
    org.opencontainers.image.title="Home Assistant App: eBUSd" \
    org.opencontainers.image.description="Transparent wrapper of john30/ebusd — a daemon for communicating with eBUS devices (heating systems)" \
    org.opencontainers.image.source="https://github.com/aghulas/ha-addons" \
    org.opencontainers.image.licenses="Apache License 2.0"

COPY run.sh ebus_sqlite.py /
RUN chmod a+x /run.sh /ebus_sqlite.py
CMD ["/run.sh"]

HEALTHCHECK --interval=10s --timeout=3s --start-period=30s \
   CMD nc -z 127.0.0.1 8888 || exit 1
