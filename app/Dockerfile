FROM quay.io/ukhomeofficedigital/dq-alpine:master

ENV USERMAP_UID 1000
ENV RUNNER_UID 1001
ENV DOCKER_UID 1002
ENV PYTHONPATH="$PYTHONPATH:/ADT"

# Setup folder structure and copy files
RUN mkdir -p \
             /tmp/ADT/scripts \
             /tmp/ADT/data/oag \
             /tmp/ADT/stage/oag \
             /tmp/ADT/quarantine/oag \
             /tmp/ADT/failed_to_parse/oag \
             /tmp/ADT/log \
             /tmp/ADT/bin \
             /ADT \
             /home/runner/.pm2 \
             /home/runner/.ssh

COPY packages.txt /tmp/ADT/scripts
COPY ecosystem.config.js /tmp/ADT/scripts
COPY docker-entrypoint.sh /
ADD bin /tmp/ADT/bin
ADD scripts /tmp/ADT/scripts

# Add user
RUN addgroup -g $RUNNER_UID runner && \
    adduser -u $USERMAP_UID -G runner -D runner && \
    addgroup -g $DOCKER_UID docker && \
    chown -R runner:runner /ADT && \
    chown -R runner:runner /home/runner/.pm2 && \
    chown -R runner:runner /home/runner/.ssh

# Install script dependencies
RUN apk add --update py-pip && \
    apk add --update py3-boto3 && \
    apk add --update py3-paramiko && \
    apk add --update py3-requests && \
    apk add --update py3-schedule && \
    apk add --update py3-cryptography

# Copy files to PVC and set permissions
RUN chmod +x docker-entrypoint.sh &&\
    /docker-entrypoint.sh


ENTRYPOINT ["tail", "-f", "/docker-entrypoint.sh"]
