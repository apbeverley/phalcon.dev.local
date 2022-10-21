ARG PHP_VERSION

FROM php:${PHP_VERSION}

ENV PHPIZE_DEPS="autoconf dpkg-dev dpkg file g++ gcc libc-dev make pkgconf re2c git"

RUN apk update && apk upgrade --available

#####################################
# install extensions:
#####################################

# intl, bz2, soap, sockets, exif, zip
RUN apk add --update --no-cache libintl libzip libzip-dev zip unzip icu icu-dev libxml2-dev bzip2-dev \
    && docker-php-ext-install intl bz2 soap sockets exif zip

# pcov psr apcu
RUN apk add --update --no-cache --virtual .build-dependencies $PHPIZE_DEPS \
        && pecl install pcov apcu psr  \
        && docker-php-ext-enable pcov apcu psr \
        && pecl clear-cache \
        && apk del .build-dependencies

## install mcrypt
RUN apk add --no-cache libmcrypt \
    && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS libmcrypt-dev \
    && yes '' | pecl install -f mcrypt \
    && docker-php-ext-enable mcrypt \
    && apk del .build-deps

# install bcmath extension
RUN docker-php-ext-install bcmath

#####################################
# Database Connections:
#####################################

# pdo, pdo_mysql, mysqli
RUN apk add --update --no-cache \
    && docker-php-ext-install pdo_mysql mysqli

#####################################
# Phalcon and tool:
#####################################

ARG INSTALL_PHALCON=false
ARG PHALCON_VERSION
ARG PHALCON_DEV_TOOLS_VERSION
ENV \
   PHALCON_VERSION=${PHALCON_VERSION} \
   PHALCON_DEV_TOOLS_VERSION=${PHALCON_DEV_TOOLS_VERSION}

RUN if [ ${INSTALL_PHALCON} = true ]; then \
    apk add --no-cache --virtual .build-deps autoconf g++ file make pcre-dev \
    && curl -LO https://github.com/phalcon/cphalcon/archive/v${PHALCON_VERSION}.tar.gz \
    && tar xzf v${PHALCON_VERSION}.tar.gz && cd cphalcon-${PHALCON_VERSION}/build && sh install  \
    && docker-php-ext-enable phalcon \
    && cd ../.. && rm -rf v${PHALCON_VERSION}.tar.gz cphalcon-${PHALCON_VERSION} \
    && apk del .build-deps \
;fi

#####################################
# Cleanup and Settings:
#####################################

RUN mv /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini \
    && echo "expose_php=0" > /usr/local/etc/php/php.ini \
    && echo "date.timezone=Europe/London" > /usr/local/etc/php/php.ini \
    && echo "memory_limit=-1" >> /usr/local/etc/php/php.ini \
    && echo "phar.readonly=0" >> /usr/local/etc/php/php.ini \
    && echo "pcov.enabled=0" >> /usr/local/etc/php/php.ini

# Change OPCache settings
RUN sed -i 's/;opcache.enable=1/opcache.enable=1/g' /usr/local/etc/php/php.ini
RUN sed -i 's/;opcache.enable_cli=0/opcache.enable_cli=1/g' /usr/local/etc/php/php.ini
RUN sed -i 's/;opcache.interned_strings_buffer=8/opcache.interned_strings_buffer=8/g' /usr/local/etc/php/php.ini
RUN sed -i 's/;opcache.max_accelerated_files=10000/opcache.max_accelerated_files=10000/g' /usr/local/etc/php/php.ini
RUN sed -i 's/;opcache.memory_consumption=128/opcache.memory_consumption=128/g' /usr/local/etc/php/php.ini
RUN sed -i 's/;opcache.save_comments=1/opcache.save_comments=1/g' /usr/local/etc/php/php.ini
RUN sed -i 's/;opcache.revalidate_freq=2/opcache.revalidate_freq=1/g' /usr/local/etc/php/php.ini

# Change user
ARG UID
ARG GID

# Clean up apk cache
RUN rm -rf /var/cache/apk/* /var/cache/apk/* /tmp/* /var/tmp/* /usr/share/doc/* /usr/share/man/*

CMD php-fpm