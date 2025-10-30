# Build a Postgres image that runs nuq.sql during initdb with Railway SSL support

ARG POSTGRES_VERSION=17
FROM postgres:${POSTGRES_VERSION}

# Set PG_MAJOR for use in pg_cron installation
ARG POSTGRES_VERSION
ENV PG_MAJOR=${POSTGRES_VERSION}

# Install pg_cron, openssl, and sudo (openssl and sudo required for Railway SSL)
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        postgresql-${PG_MAJOR}-cron \
        openssl \
        sudo; \
    rm -rf /var/lib/apt/lists/*

# Allow the postgres user to execute certain commands as root without a password
RUN echo "postgres ALL=(root) NOPASSWD: /usr/bin/mkdir, /bin/chown, /usr/bin/openssl" > /etc/sudoers.d/postgres

# Ensure pg_cron is preloaded on first startup by modifying the initdb template
# This must be set before the first server start (init scripts run after start)
RUN set -eux; \
    conf_sample="/usr/share/postgresql/${PG_MAJOR}/postgresql.conf.sample"; \
    sed -ri "s/^#?shared_preload_libraries\s*=.*/shared_preload_libraries = 'pg_cron'/" "$conf_sample"; \
    printf "\n# Added for pg_cron\ncron.database_name = 'postgres'\n" >> "$conf_sample"

# Copy Railway SSL management scripts with executable permissions
COPY --chmod=755 init-ssl.sh /docker-entrypoint-initdb.d/init-ssl.sh
COPY --chmod=755 wrapper.sh /usr/local/bin/wrapper.sh

# Copy nuq.sql so it is executed as part of the initdb sequence
COPY nuq.sql /docker-entrypoint-initdb.d/010-nuq.sql

# Use wrapper.sh as entrypoint for Railway SSL support
ENTRYPOINT ["wrapper.sh"]
CMD ["postgres", "-p", "5432", "-c", "listen_addresses=*"]