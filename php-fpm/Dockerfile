FROM php:7.4-fpm

#COPY etc/ /etc/

RUN apt-get update && apt-get install -y libxml2-dev libreadline-dev libbz2-dev zlib1g-dev libmcrypt-dev libedit-dev libcurl4-gnutls-dev libzip-dev \
    && docker-php-ext-install -j$(nproc) mysqli pdo_mysql zip bz2

COPY usr/ /usr/
