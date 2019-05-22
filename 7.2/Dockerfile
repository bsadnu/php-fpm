FROM php:7.2-fpm

LABEL maintainer="Stas Bannikov <sab.ukraine@gmail.com>"

RUN apt-get update && \
    apt-get install -y git zip && \
    apt-get install php7.2-cli php7.2-common php7.2-json php7.2-opcache php7.2-mysql php7.2-mbstring php7.2-mcrypt php7.2-zip

RUN curl --silent --show-error https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer

EXPOSE 9000

WORKDIR /app
