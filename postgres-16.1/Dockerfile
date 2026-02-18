FROM postgres:16.1

RUN apt-get update && \
    apt-get install -y postgresql-server-dev-16 build-essential git && \
    git clone --branch v0.6.3 https://github.com/pgvector/pgvector.git /pgvector && \
    cd /pgvector && make && make install && \
    rm -rf /var/lib/apt/lists/* /pgvector
