FROM php:7.4-fpm-alpine

MAINTAINER Vladyslav Shut <fred.shut.vlad@gmail.com>

ENV WORKDIR "/var/www/app"

RUN apk upgrade --update && apk --no-cache add \
    git autoconf tzdata openntpd libcurl curl-dev coreutils \
    libzip-dev \
    oniguruma-dev \   
    libmcrypt-dev freetype-dev libxpm-dev libjpeg-turbo-dev libvpx-dev \
    libpng-dev libressl-dev libxml2-dev postgresql-dev icu-dev \
    yarn php-zip

RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ --allow-untrusted gnu-libiconv
RUN export LD_PRELOAD=/usr/lib/preloadable_libiconv.so

RUN apk add --no-cache --virtual build-dependencies icu-dev libxml2-dev freetype-dev libpng-dev libjpeg-turbo-dev g++ make autoconf

RUN docker-php-ext-configure intl \
    && docker-php-ext-configure opcache \
    && docker-php-ext-configure gd --with-freetype=/usr/include/ \
    --with-jpeg=/usr/include/ --with-png=/usr/include/ \
    --with-xpm=/usr/include/

RUN docker-php-ext-install -j$(nproc) gd iconv pdo pdo_mysql pdo_pgsql curl \
    bcmath json xml xmlrpc intl opcache mbstring zip gmp

RUN mkdir -p /usr/src/php/ext/redis && curl -L https://github.com/phpredis/phpredis/archive/3.0.0.tar.gz | tar xvz -C /usr/src/php/ext/redis --strip 1 && echo 'redis' >> /usr/src/php-available-exts && docker-php-ext-install redis

# Add mcrypt
RUN yes '' | pecl install -f mcrypt
RUN echo "extension=mcrypt.so" > /usr/local/etc/php/conf.d/mcrypt.ini

# Add timezone
RUN rm -f /etc/localtime && \
    ln -s /usr/share/zoneinfo/UTC /etc/localtime && \
    "date"

# Install composer
RUN curl -sS https://getcomposer.org/installer | \
    php -- --install-dir=/usr/local/bin --filename=composer

# Cleanup
RUN rm -rf /var/cache/apk/* \
    && find / -type f -iname \*.apk-new -delete \
    && rm -rf /var/cache/apk/*

RUN mkdir -p ${WORKDIR}
RUN chown www-data:www-data -R ${WORKDIR}

WORKDIR ${WORKDIR}

EXPOSE 9000

CMD ["php-fpm"]
