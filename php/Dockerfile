ARG PHP_VERSION=7.2
FROM daocloud.io/library/php:${PHP_VERSION}-fpm



# Timezone
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo 'Asia/Shanghai' > /etc/timezone

RUN apt-get update && apt-get install -y \
    libpng-dev \
    libpq-dev \
    g++ \
    libicu-dev \
    libxml2-dev \
    htop \
    git \
    vim \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    zlib1g-dev \
    librabbitmq-dev

RUN docker-php-ext-configure intl \
    && docker-php-ext-install intl \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install zip \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install pdo_pgsql \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install soap \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install opcache \
    && docker-php-ext-install pcntl \
    && docker-php-ext-install posix \
    && docker-php-ext-install sysvsem \
    && docker-php-ext-install sysvshm \
    && docker-php-ext-install sysvmsg

# 压缩 tar -czvf ***.tar.gz
# 解压 tar -xzvf ***.tar.gz

# RUN mkdir -p /tmp/extension
RUN mkdir -p /tmp/extension
ADD spread /tmp/extension


RUN mkdir -p amqp \
    && tar -xzvf /tmp/extension/amqp-1.9.3.tgz  -C amqp --strip-components=1 \
    && ( \
    cd amqp \
    && phpize \
    && ./configure --enable-amqp \
    && make -j$(nproc) \
    && make install \
    ) \
    && rm -r amqp \
    && docker-php-ext-enable amqp

# Hiredis

RUN   mkdir -p hiredis \
    && tar -xf /tmp/extension/hiredis.tar.gz -C hiredis --strip-components=1 \
    && ( \
        cd hiredis \
        && make -j$(nproc) \
        && make install \
        && ldconfig \
    ) \
    && rm -r hiredis


RUN   mkdir -p swoole \
    && tar -xf /tmp/extension/swoole-4.1.2.tar.gz -C swoole --strip-components=1 \
    && ( \
        cd swoole \
        && phpize \
        && ./configure --enable-async-redis --enable-swoole \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r swoole \
    && docker-php-ext-enable swoole


RUN  pecl install /tmp/extension/redis-4.2.0.tgz \
    && docker-php-ext-enable redis


RUN pecl install /tmp/extension/yar-4.2.0.tgz \
 && docker-php-ext-enable yar



#composer安装
RUN echo "memory_limit=-1" > "$PHP_INI_DIR/conf.d/memory-limit.ini" \
    && echo "date.timezone=${PHP_TIMEZONE:-UTC}" > "$PHP_INI_DIR/conf.d/date_timezone.ini"
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /tmp
ENV COMPOSER_VERSION 1.6.5

RUN cp /tmp/extension/composer.phar /usr/bin/composer \
    && chmod u+x /usr/bin/composer \
    && composer self-update --clean-backups




RUN rm -r /tmp/extension

ADD php.ini /usr/local/etc/php/



# WORKDIR /var/www/html/queue.yunkezan.com

# CMD ["php", "./easyswoole", "start"]
