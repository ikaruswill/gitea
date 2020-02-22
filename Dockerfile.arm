FROM alpine:3.11
LABEL maintainer="maintainers@gitea.io"

ARG GITEA_VERSION

EXPOSE 22 3000

RUN apk --no-cache add \
    bash \
    ca-certificates \
    curl \
    gettext \
    git \
    linux-pam \
    openssh \
    s6 \
    sqlite \
    su-exec \
    tzdata

RUN addgroup \
    -S -g 1000 \
    git && \
  adduser \
    -S -H -D \
    -h /data/git \
    -s /bin/bash \
    -u 1000 \
    -G git \
    git && \
  echo "git:$(dd if=/dev/urandom bs=24 count=1 status=none | base64)" | chpasswd

ENV USER git
ENV GITEA_CUSTOM /data/gitea

VOLUME ["/data"]

ENTRYPOINT ["/usr/bin/entrypoint"]
CMD ["/bin/s6-svscan", "/etc/s6"]

# Install build dependencies (will be deleted from the image after the build)
RUN apk --no-cache --virtual .build-deps add \
    gnupg \
    rsync

# Pull docker files (sparse checkout: https://stackoverflow.com/a/13738951),
# merge them into /etc and /usr/bin/ with `rsync` and delete repository again
RUN mkdir /gitea-docker \
    && cd /gitea-docker \
    && git init \
    && git remote add -f origin https://github.com/go-gitea/gitea.git \
    && git config core.sparseCheckout true \
    && echo "docker/" >> .git/info/sparse-checkout \
    && git pull origin master \
    && rsync -av /gitea-docker/docker/root/ / \
    && rm -rf /gitea-docker

# Get gitea and verify signature
RUN mkdir -p /app/gitea \
    && gpg --keyserver keyserver.ubuntu.com --recv 7C9E68152594688862D62AF62D9AE806EC1592E2 \
    && curl -sLo /app/gitea/gitea https://github.com/go-gitea/gitea/releases/download/${GITEA_VERSION}/gitea-${GITEA_VERSION#v}-linux-arm-6 \
    && curl -sLo /app/gitea/gitea.asc https://github.com/go-gitea/gitea/releases/download/${GITEA_VERSION}/gitea-${GITEA_VERSION#v}-linux-arm-6.asc \
    && gpg --verify /app/gitea/gitea.asc /app/gitea/gitea \
    && chmod 0755 /app/gitea/gitea \
    && ln -s /app/gitea/gitea /usr/local/bin/gitea \
    && rm -rf /root/.gnupg

# Delete build dependencies
RUN apk del .build-deps
