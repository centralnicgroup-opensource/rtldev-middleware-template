# FRAME EXAMPLE (multi-service) — copy to .devcontainer/Dockerfile.
#
# Builds the dev container service referenced by docker-compose.yml. As thin as
# the single-container frame: the shared setup is in the devbase Feature, so what
# remains is whatever this stack's service genuinely needs at image level.

FROM mcr.microsoft.com/devcontainers/base:2-ubuntu-24.04

ENV DEBIAN_FRONTEND=noninteractive
ENV TERM=xterm-256color

# Services this container runs itself (a web server, for instance) are installed
# here; the devcontainer features cover language runtimes, and devbase covers the
# shared shell tooling.
#
# hadolint ignore=DL3008
# RUN apt-get update \
#     && apt-get -y install --no-install-recommends \
#        {{SYSTEM_PACKAGES}} \
#     && apt-get -y autoremove \
#     && apt-get -y clean \
#     && rm -rf /var/lib/apt/lists/*

# Image-level service configuration, if any.
# COPY supporting_files/configuration/apache2/ /etc/apache2/
