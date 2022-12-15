FROM composer:2.3 as vendor

WORKDIR /app

COPY composer.json composer.json
COPY composer.lock composer.lock

RUN composer install \
    --ignore-platform-reqs \
    --no-interaction \
    --no-plugins \
    --no-scripts \
    --prefer-dist \
    --optimize-autoloader \
    --no-dev

FROM php:8.2-fpm as fpm-prod

RUN docker-php-ext-enable opcache \
    && docker-php-ext-install sockets \
    && apt-get --allow-releaseinfo-change update \
    && apt-get install -y --no-install-recommends \
        libcurl4-openssl-dev \
        pkg-config \
        libssl-dev \
    && apt-get clean autoclean \
    && apt-get autoremove --yes \
    && rm -rf /var/lib/{apt,dpkg,cache,log}/ \
    && printf 'post_max_size = 10M\n' >> /usr/local/etc/php/conf.d/upload-files-php.ini \
    && printf 'upload_max_filesize = "10M"\n' >> /usr/local/etc/php/conf.d/upload-files-php.ini \
    && printf 'memory_limit="256M"\n' >> /usr/local/etc/php/conf.d/z_memory.ini

COPY . /var/www/html
COPY --from=vendor /app/vendor/ /var/www/html/vendor
COPY --from=vendor /usr/bin/composer /usr/bin/composer

FROM fpm-prod as fpm-dev

WORKDIR /var/www/html

RUN apt-get update --allow-releaseinfo-change \
    && apt-get install -y --no-install-recommends git zip unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && composer install \
       --ignore-platform-reqs \
       --no-interaction \
       --no-plugins \
       --no-scripts \
       --prefer-dist
