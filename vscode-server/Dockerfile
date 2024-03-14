FROM python:3.8

ENV VERSION=4.22.0
ENV DOMAIN=localhost

CMD ["bash"]

RUN apt update \
    && apt install -y curl dumb-init git git-lfs htop locales lsb-release man-db nano openssh-client procps sudo vim-tiny wget zsh && git lfs install && rm -rf /var/lib/apt/lists/*
RUN sed -i "s/# en_US.UTF-8/en_US.UTF-8/" /etc/locale.gen   && locale-gen # buildkit

ENV LANG=en_US.UTF-8

RUN adduser --gecos '' --disabled-password sinredemption   && echo "coder ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/nopasswd # buildkit
RUN ARCH="$(dpkg --print-architecture)"   && curl -fsSL "https://github.com/boxboat/fixuid/releases/download/v0.6.0/fixuid-0.6.0-linux-$ARCH.tar.gz" | tar -C /usr/local/bin -xzf -   && chown root:root /usr/local/bin/fixuid   && chmod 4755 /usr/local/bin/fixuid   && mkdir -p /etc/fixuid   && printf "user: sinredemption\ngroup: sinredemption\n" > /etc/fixuid/config.yml 

RUN curl -fOL https://github.com/coder/code-server/releases/download/v$VERSION/code-server_${VERSION}_amd64.deb
RUN dpkg -i code-server_${VERSION}_amd64.deb

RUN mkdir /certs && cd /certs \
    && export PASSPHRASE=$(head -c 500 /dev/urandom | tr -dc a-z0-9A-Z | head -c 128; echo) \
    && openssl genrsa -des3 -out $DOMAIN.key -passout env:PASSPHRASE 2048 \
    && export subj='/C=ZA/ST=WC/O=Org/localityName=CT/commonName=localhost/organizationalUnitName=OrgUnit/emailAddress=root@localhost/' \
    && openssl req -new -batch -subj "${subj}" -key $DOMAIN.key -out $DOMAIN.csr -passin env:PASSPHRASE \
    && cp $DOMAIN.key $DOMAIN.key.org \
    && openssl rsa -in $DOMAIN.key.org -out $DOMAIN.key -passin env:PASSPHRASE \
    && openssl x509 -req -days 3650 -in $DOMAIN.csr -signkey $DOMAIN.key -out $DOMAIN.crt

RUN chmod 4755 /certs/$DOMAIN.key && chmod 4755 /certs/$DOMAIN.csr

COPY /docker-entrypoint.sh /usr/bin/entrypoint.sh

ENV ENTRYPOINTD=/entrypoint.d
EXPOSE 8080/tcp
USER 1000
ENV USER=sinredemption

ENTRYPOINT [ "/usr/bin/entrypoint.sh","--bind-addr","0.0.0.0:8080","."]