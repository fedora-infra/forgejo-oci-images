FROM quay.io/fedora/fedora:rawhide

RUN dnf install -y git-core nodejs npm golang && dnf clean all


ARG GOPROXY
ENV GOPROXY=${GOPROXY:-direct}

ARG RELEASE_VERSION
ENV TAGS="bindata timetzdata $TAGS"
ARG CGO_EXTRA_CFLAGS

ENV CGO_ENABLED=1


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

RUN wget -qO- https://codeberg.org/forgejo/forgejo/archive/v10.0/forgejo.tar.gz | tar -xzvf -

COPY ./forgejo ${GOPATH}/src/code.gitea.io/gitea
WORKDIR ${GOPATH}/src/code.gitea.io/gitea

RUN make clean
RUN make frontend
RUN go build contrib/environment-to-ini/environment-to-ini.go && xx-verify environment-to-ini
RUN make RELEASE_VERSION=$RELEASE_VERSION go-check generate-backend static-executable && xx-verify gitea

# Copy local files
COPY docker/rootless /tmp/local


# Set permissions
RUN chmod 755 /tmp/local/usr/local/bin/docker-entrypoint.sh \
              /tmp/local/usr/local/bin/docker-setup.sh \
              /tmp/local/usr/local/bin/gitea \
              /go/src/code.gitea.io/gitea/gitea \
              /go/src/code.gitea.io/gitea/environment-to-ini
RUN chmod 644 /go/src/code.gitea.io/gitea/contrib/autocompletion/bash_autocomplete

