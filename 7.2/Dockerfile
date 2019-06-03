FROM php:7.2-fpm

MAINTAINER Stas Bannikov <sab.ukraine@gmail.com>

RUN apt-get update && apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libpq-dev \
    g++ \
    libicu-dev \
    libxml2-dev \
    git \
    vim \
    libmcrypt-dev \
    zlib1g-dev \
    libmagickwand-dev --no-install-recommends \
    libzip-dev \
    procps \
    librabbitmq-dev \
    libssh-dev \
    libxrender1 \
    libfontconfig1 \
    libpcre3-dev \
    libbz2-dev \
    zip \
    unzip \
    libxslt-dev

RUN pecl install mcrypt-1.0.2

RUN docker-php-ext-configure intl \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install intl \
    && docker-php-ext-install zip \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install pdo_pgsql \
    && docker-php-ext-install soap \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install opcache \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install bcmath \
    && docker-php-ext-install pcntl \
    && docker-php-ext-install sockets \
    && pecl install imagick  \
    && docker-php-ext-enable imagick \
    && pecl install apcu \
    && docker-php-ext-install iconv \
    && docker-php-ext-enable mcrypt

RUN pecl install amqp \
    docker-php-ext-enable amqp

RUN apt-get install -y autoconf pkg-config libssl-dev
RUN pecl install mongodb

RUN docker-php-ext-enable mongodb

RUN pecl install ds

RUN docker-php-ext-enable ds

ENV PHPREDIS_VERSION 3.1.4

RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz  \
    && mkdir /tmp/redis \
    && tar -xf /tmp/redis.tar.gz -C /tmp/redis \
    && rm /tmp/redis.tar.gz \
    && ( \
    cd /tmp/redis/phpredis-$PHPREDIS_VERSION \
    && phpize \
    && ./configure \
    && make -j$(nproc) \
    && make install \
    ) \
    && rm -r /tmp/redis \
    && docker-php-ext-enable redis

RUN sed -i -e 's/listen.*/listen = 0.0.0.0:9000/' /usr/local/etc/php-fpm.conf

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN usermod -u 1000 www-data

CMD ["php-fpm"]

EXPOSE 9000

WORKDIR /app

ENV BLACKFIRE_LOG_LEVEL 1
ENV BLACKFIRE_LOG_FILE /tmpfs/logs/blackfire.log
ENV BLACKFIRE_SOCKET unix:///var/run/blackfire/agent.sock

RUN apt-get install -y \
    wget \
    gnupg2

RUN wget -q -O - https://packages.blackfire.io/gpg.key | apt-key add - && \
    echo "deb http://packages.blackfire.io/debian any main" | tee /etc/apt/sources.list.d/blackfire.list

RUN apt-get update

RUN apt-get install -y \
    blackfire-agent \
    blackfire-php

ENV current_os=linux
RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/$current_os/amd64/$version \
    && mkdir -p /tmp/blackfire \
    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp/blackfire \
    && mv /tmp/blackfire/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
    && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/conf.d/blackfire.ini \
    && rm -rf /tmp/blackfire /tmp/blackfire-probe.tar.gz

COPY blackfire-agent.ini /etc/blackfire/agent
COPY blackfire-php.ini /etc/php/7.2/fpm/conf.d/92-blackfire-config.ini
COPY blackfire-php.ini /etc/php/7.2/cli/conf.d/92-blackfire-config.ini

COPY blackfire-run.sh /blackfire-run.sh

ENTRYPOINT ["/bin/bash", "/blackfire-run.sh"]
