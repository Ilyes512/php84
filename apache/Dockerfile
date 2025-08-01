# Latest version of PHP base image: https://hub.docker.com/_/php?tab=tags
FROM php:8.4.10-apache-bookworm AS runtime

ARG UNIQUE_ID_FOR_CACHEFROM=runtime

# Latest version of event-extension: https://pecl.php.net/package/event
ARG PHP_EVENT_VERSION=3.1.4
# Latest version of igbinary-extension: https://pecl.php.net/package/igbinary
ARG PHP_IGBINARY_VERSION=3.2.16
# Latest version of redis-extension: https://pecl.php.net/package/redis
ARG PHP_REDIS_VERSION=6.2.0
# Latest version of memcached-extension: https://pecl.php.net/package/memcached
ARG PHP_MEMCACHED_VERSION=3.3.0
# Latest version of amqp-extension: https://pecl.php.net/package/amqp
ARG PHP_AMQP_VERSION=2.1.2

ENV SMTPHOST=mail
ENV SMTPEHLO=localhost

WORKDIR /var/www

RUN apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        openssl \
        curl \
        msmtp-mta \
        # Dependency of the PHP intl-extension
        libicu72 \
        # Dependency of the PHP gd-extension
        libpng16-16 \
        libwebp7 \
        libjpeg62-turbo \
        libfreetype6 \
        # Dependency of PHP zip-extension
        libzip4 \
        # Dependency of PHP event-extension
        libevent-2.1-7 \
        libevent-openssl-2.1-7 \
        libevent-extra-2.1-7 \
        # Dependency of PHP pdo_pgsql-extension
        libpq5 \
        # Dependency of PHP amqp-extension
        librabbitmq4 \
        # Dependency of PHP xsl-extension
        libxslt1.1 \
        # Dependency of PHP memcached-extension
        libmemcached11 \
        zlib1g \
    # Install packages that are needed for building PHP extensions
    && apt-get install --assume-yes --no-install-recommends \
        $PHPIZE_DEPS \
        # Dependency of the PHP intl-extension
        libicu-dev \
        # Dependencies of PHP gd-extension
        libpng-dev \
        libwebp-dev \
        libjpeg62-turbo-dev \
        libfreetype6-dev \
        # Dependency of PHP zip-extension
        libzip-dev \
        # Dependency of PHP event-extension
        libevent-dev \
        libssl-dev \
        # Dependency of PHP soap-extension
        libxml2-dev \
        # Dependency of PHP pdo_pgsql-extension
        libpq-dev \
        # Dependency of PHP amqp-extension
        librabbitmq-dev \
        # Dependency of PHP xsl-extension
        libxslt1-dev \
        # Dependency of PHP memcached-extension
        libmemcached-dev \
        zlib1g-dev \
    # Configure PHP gd-extension
    && docker-php-ext-configure gd \
        --enable-gd \
        --with-jpeg \
        --with-freetype \
        --with-webp \
    # Install PHP extensions
    && docker-php-ext-install -j "$(nproc --all)" \
        pdo_mysql \
        pdo_pgsql \
        intl \
        opcache \
        pcntl \
        gd \
        bcmath \
        zip \
        soap \
        xsl \
        # Dependency of PHP event-extension
        sockets \
    && pecl install "event-$PHP_EVENT_VERSION" \
    # Optional dependency of PHP redis-extension
    && pecl install "igbinary-$PHP_IGBINARY_VERSION" \
    && pecl install --configureoptions 'enable-redis-igbinary="yes"' "redis-$PHP_REDIS_VERSION" \
    && pecl install --configureoptions 'enable-memcached-igbinary="yes"' "memcached-$PHP_MEMCACHED_VERSION" \
    && pecl install "amqp-$PHP_AMQP_VERSION" \
    && docker-php-ext-enable --ini-name docker-php-ext-zz-custom.ini \
        event \
        igbinary \
        redis \
        memcached \
        amqp \
    && cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini" \
    # Purge packages that where only needed for building php extensions
    && apt-get purge --assume-yes \
        $PHPIZE_DEPS \
        libicu-dev \
        libpng-dev \
        libwebp-dev \
        libjpeg62-turbo-dev \
        libfreetype6-dev \
        libzip-dev \
        libevent-dev \
        libssl-dev \
        libxml2-dev \
        libpq-dev \
        librabbitmq-dev \
        libxslt1-dev \
        libmemcached-dev \
        zlib1g-dev \
    # Cleanup
    && rm -rf /var/www/* \
    && apt-get autoremove --assume-yes \
    && apt-get clean --assume-yes \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

COPY files /

FROM runtime AS builder

ARG UNIQUE_ID_FOR_CACHEFROM=builder

# Latest version of Phive: https://api.github.com/repos/phar-io/phive/releases/latest
ARG PHIVE_VERSION=0.16.0
# Latest version of Composer: https://getcomposer.org/download
ARG COMPOSER_VERSION=2.8.10
# Latest version of Xdebug: https://github.com/xdebug/xdebug/tags or https://pecl.php.net/package/xdebug
ARG XDEBUG_VERSION=3.4.5
# Latest version of pcov: https://github.com/krakjoe/pcov/tags or https://pecl.php.net/package/pcov
ARG PCOV_VERSION=1.0.12

RUN apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        # Needed for xdebug extension configuration
        $PHPIZE_DEPS \
        vim \
        git \
        unzip \
        sqlite3 \
        wait-for-it \
        # Needed for phive:
        gnupg \
    # Install Phive
    && curl -fsSLo /usr/local/bin/phive "https://github.com/phar-io/phive/releases/download/$PHIVE_VERSION/phive-$PHIVE_VERSION.phar" \
    && curl -fsSLo /tmp/phive.phar.asc "https://github.com/phar-io/phive/releases/download/$PHIVE_VERSION/phive-$PHIVE_VERSION.phar.asc" \
    && gpg --keyserver keys.openpgp.org --recv-keys 0x9D8A98B29B2D5D79 \
    && gpg --verify /tmp/phive.phar.asc /usr/local/bin/phive \
    && chmod +x /usr/local/bin/phive \
    && phive update-repository-list \
    # Install Composer using Phive
    && phive install --global composer:$COMPOSER_VERSION --trust-gpg-keys CBB3D576F2A0946F \
    && rm -rf /root/.phive \
    # Install Xdebug PHP extension
    && pecl install "xdebug-$XDEBUG_VERSION" \
    && docker-php-ext-enable xdebug \
    # Install pcov PHP extension
    && pecl install "pcov-$PCOV_VERSION" \
    && docker-php-ext-enable pcov \
    && cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini" \
    # Cleanup
    && apt-get purge --assume-yes $PHPIZE_DEPS \
    && apt-get autoremove --assume-yes \
    && apt-get clean --assume-yes \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

FROM builder AS builder_nodejs

ARG TARGETARCH

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG UNIQUE_ID_FOR_CACHEFROM=builder_nodejs

# Latest version of Node.js: https://nodejs.org
ARG NODE_MAJOR=24

RUN apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        gcc \
        g++ \
        make \
    && mkdir -p /usr/share/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /usr/share/keyrings/nodesource.gpg \
    && chmod 644 /usr/share/keyrings/nodesource.gpg \
    && echo "deb [arch=$TARGETARCH signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
    && echo "Package: nodejs" | tee /etc/apt/preferences.d/nodejs > /dev/null \
    && echo "Pin: origin deb.nodesource.com" | tee -a /etc/apt/preferences.d/nodejs > /dev/null \
    && echo "Pin-Priority: 600" | tee -a /etc/apt/preferences.d/nodejs > /dev/null \
    && apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        nodejs \
    && npm uninstall --global npm \
    && corepack install --global npm@11.x yarn@4.x pnpm@10.x \
    && corepack enable npm yarn pnpm \
    && apt-get autoremove --assume-yes \
    && apt-get clean --assume-yes \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*
