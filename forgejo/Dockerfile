FROM quay.io/fedora/fedora:rawhide

RUN dnf install -y git-core wget nodejs npm golang && dnf clean all


ARG GOPROXY
ENV GOPROXY=${GOPROXY:-direct}
ARG GOPATH
ENV GOPATH=""

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

RUN mkdir -p ${GOPATH}/src/code.gitea.io/gitea

ADD https://codeberg.org/forgejo/forgejo/archive/v10.0/forgejo.tar.gz ${GOPATH}/

RUN tar -xvzf ${GOPATH}/forgejo.tar.gz -C ${GOPATH}/src/code.gitea.io/gitea/ --strip-components=1

WORKDIR ${GOPATH}/src/code.gitea.io/gitea

RUN ls -la

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

RUN mkdir -p /var/lib/gitea /etc/gitea
RUN chown git:git /var/lib/gitea /etc/gitea

COPY --from=build-env /tmp/local /
RUN cd /usr/local/bin ; ln -s gitea forgejo
COPY --from=build-env --chown=root:root /go/src/code.gitea.io/gitea/gitea /app/gitea/gitea
RUN ln -s /app/gitea/gitea /app/gitea/forgejo-cli
COPY --from=build-env --chown=root:root /go/src/code.gitea.io/gitea/environment-to-ini /usr/local/bin/environment-to-ini
COPY --from=build-env /go/src/code.gitea.io/gitea/contrib/autocompletion/bash_autocomplete /etc/profile.d/gitea_bash_autocomplete.sh

#git:git
USER 1000:1000
ENV GITEA_WORK_DIR=/var/lib/gitea
ENV GITEA_CUSTOM=/var/lib/gitea/custom
ENV GITEA_TEMP=/tmp/gitea
ENV TMPDIR=/tmp/gitea

# Legacy config file for backwards compatibility
# TODO: remove on next major version release
ENV GITEA_APP_INI_LEGACY=/etc/gitea/app.ini

ENV GITEA_APP_INI=${GITEA_CUSTOM}/conf/app.ini
ENV HOME="/var/lib/gitea/git"
VOLUME ["/var/lib/gitea", "/etc/gitea"]
WORKDIR /var/lib/gitea

ENTRYPOINT ["/usr/bin/dumb-init", "--", "/usr/local/bin/docker-entrypoint.sh"]
CMD []

