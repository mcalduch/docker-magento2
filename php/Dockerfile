FROM php:7.2-fpm-alpine as magento2_php

# Install Magento requirements
RUN \
    apk add --no-cache --virtual .persistent-deps \
        freetype-dev \
        git \
        icu-libs \
        libjpeg-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libxml2-dev \
        libxml2-utils \
        libxslt-dev \
        openssh-client \
        patch \
        perl \
        ssmtp \
        yarn && \
    apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        icu-dev && \
    docker-php-ext-configure bcmath && \
    docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-install -j$(nproc) \
        bcmath \
        intl \
        gd \
        opcache \
        pdo_mysql \
        soap \
        xsl \
        zip && \
    yes "" | pecl install apcu redis && \
    docker-php-ext-enable apcu redis && \
    perl -pi -e "s/mailhub=mail/mailhub=maildev/" /etc/ssmtp/ssmtp.conf && \
    perl -pi -e "s|;pm.status_path = /status|pm.status_path = /php_fpm_status|g" /usr/local/etc/php-fpm.d/www.conf && \
    yarn global add grunt-cli && \
    apk del .build-deps

## Install Composer globally
ENV COMPOSER_ALLOW_SUPERUSER 1
RUN \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    composer global require "hirak/prestissimo:dev-master" "n98/magerun2:dev-master" \
        --no-suggest --optimize-autoloader --classmap-authoritative
ENV PATH "${PATH}:/root/.composer/vendor/bin"

# Create directories where Magento will write
RUN \
    mkdir -p \
        /var/www/html/generated \
        /var/www/html/pub/media \
        /var/www/html/pub/static \
        /var/www/html/var/cache \
        /var/www/html/var/page_cache \
        /var/www/html/var/session \
        /var/www/html/var/view_preprocessed \
        /var/www/html/vendor && \
    chown -R www-data:root /var/www/html

# Install custom entrypoint
COPY entrypoint.sh /usr/local/bin/docker-custom-entrypoint
RUN chmod 777 /usr/local/bin/docker-custom-entrypoint
CMD ["php-fpm"]
ENTRYPOINT ["docker-custom-entrypoint"]

# ======================================================================================================================
FROM magento2_php as magento2_php_blackfire
RUN \
    curl -sS https://packages.blackfire.io/binaries/blackfire-php/1.23.1/blackfire-php-alpine_amd64-php-72.so \
        --output $(php -r "echo ini_get('extension_dir');")/blackfire.so && \
    docker-php-ext-enable blackfire
# ======================================================================================================================

# ======================================================================================================================
FROM magento2_php as magento2_php_xdebug
RUN \
    apk add --no-cache --virtual .build-deps $PHPIZE_DEPS && \
    yes "" | pecl install xdebug && \
    docker-php-ext-enable xdebug
# ======================================================================================================================
