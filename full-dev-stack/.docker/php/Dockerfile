FROM php:fpm-alpine
#FROM php:7.2-fpm-alpine

LABEL maintainer="Petr Zivny | Totea (zivny@totea.cz)"

RUN apk update && apk upgrade                       && \
    apk add --no-cache --virtual .build-dependencies   \
        $PHPIZE_DEPS                                && \
    \
    apk add --no-cache \
        # 32 kB The simplest pid 1 (init) process reaps forked child processes correctly, if it doesn't then bad things happen like resources get leaked and zombies appear
        tini                            \
        # 68 kB GNU gettext runtime library. A set of tools that provides a framework to help other GNU packages produce multi-lingual messages
        libintl                         \
        # 1.14 MB Collection of tools and libraries for many image formats (development files). Imagick is a native php extension to create and modify images using the ImageMagick API.
#       imagemagick-dev                 \
    && apk del .build-dependencies && rm -rf /tmp/* /var/cache/apk/*

# ZIP
# Enables to transparently read or write ZIP compressed archives and the files inside them. $zip = new ZipArchive();
RUN apk --update add libzip-dev                 && \
    docker-php-ext-configure zip --with-libzip  && \
    docker-php-ext-install zip                  && \
    rm -rf /var/cache/apk/*

# OPcache
RUN apk update                                          && \
    docker-php-ext-configure opcache --enable-opcache   && \
    docker-php-ext-install opcache                      && \
    rm -rf /var/cache/apk/*
COPY .docker/php/conf.d/00-opcache-recommended.ini /usr/local/etc/php/conf.d/00-opcache-recommended.ini
RUN if [ "x$APP_ENV" = "xprod" ]; then sed -i '/opcache.validate_timestamps=0/s/^# *//' /usr/local/etc/php/conf.d/00-opcache-recommended.ini; fi

# APCu
RUN apk --update add --virtual .build-dependencies autoconf g++ make    && \
    pecl install apcu                                                   && \
    docker-php-ext-enable apcu                                          && \
    apk del .build-dependencies && rm -rf /var/cache/apk/*
COPY .docker/php/conf.d/00-apcu-recommended.ini /usr/local/etc/php/conf.d/00-apcu-recommended.ini

# PDO MySQL
# Driver to enable access from PHP to MySQL databases. Implements PDO (doesn't need it). $dbh = new PDO($dsn, $user, $password);
RUN apk update                                                          && \
    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" pdo_mysql   && \
    rm -rf /var/cache/apk/*

## PDO PostgreSQL
## Driver to enable access from PHP to PostgreSQL databases. Implements PDO (doesn't need it). $dbh = new PDO("pgsql:dbname=$dbname;host=$host", $dbuser, $dbpass);
#RUN apk --update add postgresql-dev                                     && \
#    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" pdo_pgsql   && \
#    rm -rf /var/cache/apk/*

## MySQLi
## Allows to access the functionality provided by MySQL 4.1 and above. $mysqli = new mysqli('127.0.0.1', 'your_user', 'your_pass', 'sakila');
#RUN apk update                                                      && \
#    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" mysqli  && \
#    rm -rf /var/cache/apk/*

# Redis
ARG PHPREDIS_VERSION="${PHPREDIS_VERSION:-4.1.1}"
ENV PHPREDIS_VERSION="${PHPREDIS_VERSION}"
ADD https://github.com/phpredis/phpredis/archive/${PHPREDIS_VERSION}.tar.gz /tmp/
RUN apk update && \
    docker-php-source extract                                       && \
    tar xfz /tmp/${PHPREDIS_VERSION}.tar.gz  -C /tmp                && \
    mv /tmp/phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis       && \
    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" redis   && \
    docker-php-source delete        && \
    rm -rf /tmp/* /var/cache/apk/*

## Memcached
#RUN apk --update add --virtual .build-dependencies git autoconf g++ make zlib-dev                               && \
#    apk --update add libmemcached-dev                                                                           && \
#    docker-php-source extract                                                                                   && \
#    git clone https://github.com/php-memcached-dev/php-memcached.git /usr/src/php/ext/memcached/ 2&> /dev/null  && \
#    docker-php-ext-configure memcached                                                                          && \
#    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" memcached                                           && \
#    apk del .build-dependencies                                                                                 && \
#    docker-php-source delete                                                                                    && \
#    rm -rf /var/cache/apk/*

# Intl
# A wrapper for ICU library (38 MB), enabling various locale-aware operations (requires ICU library >4.0), (for PHP >7.4 requires >5.0.1) $formatter = new NumberFormatter('en_US', NumberFormatter::DECIMAL);
RUN apk --update add icu-dev                                        && \
    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" intl    && \
    rm -rf /var/cache/apk/*

## Iconv
## Contains an interface to iconv character set. Can turn a string represented by a local character set into the one represented by another character set. iconv_* functions.
## Iconv is already part of the alpine image, but doesn't work properly without gnu-libiconv
#ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php
#RUN apk add gnu-libiconv --update-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing/ --allow-untrusted && \
#    rm -rf /tmp/* /var/cache/apk/*



## XSL
## Implements the XSL standard, performing XSLT transformations (requires libxml and libxslt). $proc = new XSLTProcessor();
#RUN apk --update add libxml2-dev libxslt-dev                    && \
#    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" xsl && \
#    rm -rf /var/cache/apk/*

## GMP functions allow for arbitrary-length integers to be worked with using the GNU MP library (requires GMP >2). gmú_* functions.
#RUN apk --update add gmp gmp-dev                                && \
#    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" gmp && \
#    rm -rf /var/cache/apk/*

## bcmath
## Binary Calculator for arbitrary precision mathematics. bc* functions.
#RUN apk update                                                      && \
#    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" bcmath  && \
#    rm -rf /var/cache/apk/*

##SOAP
## Can be used to write SOAP Servers and Clients (requires the libxml).
#RUN apk --update add libxml2-dev                                    && \
#    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" soap    && \
#    rm -rf /var/cache/apk/*

## GD library
## (requires freetype, libpng and libjpeg to compile)
#RUN apk --update add                                                                               \
#        # 1.34 MB TrueType font rendering library (development files).
#        freetype-dev                                                                               \
#        # 1.41 MB Accelerated baseline JPEG compression and decompression library (development files). Generally 2-6x as fast as libjpeg.
#        libjpeg-turbo-dev                                                                          \
#        # 488 kB Portable Network Graphics library (development files).
#        libpng-dev                                                                              && \
#    docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
#    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" gd                                  && \
#    rm -rf /var/cache/apk/*


ARG APP_ENV

# Development environment only (or fresh install)
RUN if [ "x$APP_ENV" = "xdev" ] || [ "x$APP_ENV" = "x" ]; then                      \
    mv /usr/local/etc/php/php.ini-development /usr/local/etc/php/conf.d/php.ini;    \
    # Xdebug
    apk --update add --virtual .build-dependencies $PHPIZE_DEPS;                    \
    pecl install xdebug;                                                            \
    docker-php-ext-enable xdebug;                                                   \
    apk del .build-dependencies;                                                    \
    rm -rf /tmp/* /var/cache/apk/*;                                                 \
    fi

# Production environment only
RUN if [ "x$APP_ENV" = "xprod" ]; then                                      \
    mv /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini;    \
    fi

# php.ini
RUN echo "expose_php=off" >> /usr/local/etc/php/php.ini
#RUN echo "date.timezone = "Europe/Prague"" >> /usr/local/etc/php/php.ini

CMD ["php-fpm"]