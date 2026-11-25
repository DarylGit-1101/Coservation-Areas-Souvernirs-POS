FROM php:8.4-fpm-alpine

# Install runtime dependencies, PHP extensions (including intl for Filament & phpredis), then remove build deps
RUN apk add --no-cache \
    git \
    curl \
    icu-dev \
    libpng-dev \
    libzip-dev \
    zip \
    unzip \
    postgresql-dev \
    && apk add --no-cache --virtual .build-deps \
    linux-headers \
    $PHPIZE_DEPS \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && docker-php-ext-install -j$(nproc) pdo pdo_pgsql bcmath intl pcntl sockets gd zip opcache \
    && apk del .build-deps

# Default OPcache settings (override with OPCACHE_VALIDATE_TIMESTAMPS=0 in production)
ENV OPCACHE_VALIDATE_TIMESTAMPS=1 \
    OPCACHE_REVALIDATE_FREQ=2

# Copy PHP, OPcache, and PHP-FPM pool configurations
COPY docker/php/php.ini $PHP_INI_DIR/conf.d/99-pos.ini
COPY docker/php/opcache.ini $PHP_INI_DIR/conf.d/opcache.ini
COPY docker/php/www.conf /usr/local/etc/php-fpm.d/zz-pos.conf

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/backend

EXPOSE 9000
CMD ["php-fpm"]
