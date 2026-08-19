# Development image for this repository.
#
# Deliberately thin. The package installs, timezone, zsh configuration and
# commitizen setup that used to live in every repository's Dockerfile are now in
# the devbase Feature, which is why this file has almost nothing left to do.
#
# Add a layer here only for something the Feature cannot express: a system
# library a native extension needs to compile, a runtime the devcontainer
# features do not package, an image-level config file. If what you are about to
# add is shared behaviour rather than this repository's own, it belongs in the
# Feature instead.

FROM mcr.microsoft.com/devcontainers/base:2-ubuntu-24.04

# No ENV DEBIAN_FRONTEND here on purpose. Set at image level it persists into the
# running container, so an interactive `apt` in a shell session also loses its
# frontend. The devcontainer Features export it themselves for their own installs,
# so nothing needs it globally. A layer below that does, sets it for that layer:
#
#   RUN DEBIAN_FRONTEND=noninteractive apt-get update && ...
ENV TERM=xterm-256color

# Example of a repository-specific system package. Uncomment and adjust; delete
# the whole layer if this repository needs none — as this one does.
#
# hadolint ignore=DL3008
# RUN DEBIAN_FRONTEND=noninteractive apt-get update \
#     && apt-get -y install --no-install-recommends \
#        libfoo-dev \
#     && apt-get -y autoremove \
#     && apt-get -y clean \
#     && rm -rf /var/lib/apt/lists/*
