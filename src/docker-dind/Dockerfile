FROM docker:dind
RUN apk update && apk add --no-cache py-pip \
    python3-dev libffi-dev openssl-dev gcc \
    curl wget \
    libc-dev make && \
    pip install --upgrade pip cryptography==2.8 docker-compose
