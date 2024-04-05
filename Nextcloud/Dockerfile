FROM nextcloud:latest
RUN set -ex; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    ffmpeg \
    libbz2-dev
RUN docker-php-ext-install bz2
