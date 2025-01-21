FROM quay.io/fedora/fedora:rawhide

RUN dnf install -y git-core nodejs npm golang && dnf clean all

LABEL maintainer="codeberg@fedoraproject.org" \
      org.opencontainers.image.authors="Forgejo on Fedora" \
      org.opencontainers.image.url="https://forgejo.fedoraproject.org" \
      org.opencontainers.image.documentation="https://codeberg.org/fedora/oci-image-definitions" \
      org.opencontainers.image.source="https://codeberg.org/fedora/oci-image-definitions" \
      org.opencontainers.image.version="${RELEASE_VERSION}" \
      org.opencontainers.image.vendor="Forgejo" \
      org.opencontainers.image.licenses="GPL-3.0-or-later" \
      org.opencontainers.image.title="Forgejo. Beyond coding. We forge." \
      org.opencontainers.image.description="Forgejo for distgit"
