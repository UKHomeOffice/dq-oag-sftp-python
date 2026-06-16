FROM alpine:3.18.7

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
    apk add --update py3-cryptography && \
    apk add --update py3-psycopg2

# Install PM2
RUN apk add --update nodejs npm
#RUN npm install -g pm2 && \
#    npm update -g pm2

# ─────────────────────────────────────────────────────────────────
# FIX CVEs & OPTIMIZE SIZE: Universal dependency override & cleanup
# ─────────────────────────────────────────────────────────────────
RUN mkdir -p /tmp/npm-patch && cd /tmp/npm-patch && \
    # 1. Create a root manifest enforcing secure versions across the entire build tree
    echo '{\
      "overrides": {\
        "systeminformation": "5.31.6",\
        "lodash": "4.18.0",\
        "ws": "8.20.1",\
        "diff": "8.0.3",\
        "serialize-javascript": "7.0.5"\
      }\
    }' > package.json && \
    # 2. Install PM2 and required packages globally using the strict overrides config
    # (Removed redundant 'npm update' command which bypassed the overrides file)
    npm install -g pm2 js-yaml@4.1.1 lodash@4.18.0 systeminformation@5.31.6 && \
    # 3. ─── CLEANUP SECTOR (Enforces strict size limits) ───
    cd / && rm -rf /tmp/npm-patch && \
    npm cache clean --force && \
    rm -rf /root/.npm/_cacache && \
    rm -rf /root/.node-gyp && \
    # Strip optional testing/doc folders brought in by npm
    find "$(npm root -g)" -type d \( -name "test" -o -name "doc" -o -name "docs" \) -exec rm -rf {} + 2>/dev/null || true

RUN apk update --quiet \
    && apk upgrade --quiet

VOLUME ["/ADT"]
RUN chown -R runner:runner /ADT

RUN chown runner:runner /docker-entrypoint.sh && \
chmod +x /docker-entrypoint.sh

USER ${USERMAP_UID}

WORKDIR /ADT

ENTRYPOINT ["sh", "/docker-entrypoint.sh"]

# Start PM2
CMD pm2-docker /ADT/scripts/ecosystem.config.js  -- --config $SSH_REMOTE_HOST_MAYTECH $SSH_REMOTE_USER_MAYTECH $SSH_PRIVATE_KEY \
    $SSH_LANDING_DIR $S3_BUCKET_NAME $S3_ACCESS_KEY_ID $S3_SECRET_ACCESS_KEY \
    $CLAMAV_URL $CLAMAV_PORT $OAG_RDS_HOST $OAG_RDS_DATABASE $OAG_RDS_USERNAME $OAG_RDS_PASSWORD \
    $OAG_RDS_TABLE $SLACK_WEBHOOK

# Save PM2 configuration
RUN pm2 save

RUN node --version
RUN npm --version
