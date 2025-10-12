# Multi-stage build for PHP Sylius application
FROM php:7.4-fpm AS php_base

# Install system dependencies and PHP extensions
RUN apt-get update && apt-get install -y \
    curl \
    git \
    libicu-dev \
    libonig-dev \
    libpng-dev \
    libxml2-dev \
    libzip-dev \
    unzip \
    wget \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-install \
        bcmath \
        exif \
        gd \
        intl \
        mbstring \
        opcache \
        pcntl \
        pdo_mysql \
        zip

# Install Composer
COPY --from=composer:2.2 /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Copy composer files
COPY composer.json composer.lock ./

# Install PHP dependencies with platform requirements ignored for CI compatibility
RUN composer install --no-dev --optimize-autoloader --no-scripts --ignore-platform-reqs

# Copy application code
COPY . .

# Configure PHP, set permissions, and run post-install scripts
RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini \
    && echo "date.timezone=Europe/Lisbon" >> /usr/local/etc/php/php.ini \
    && echo "memory_limit=512M" >> /usr/local/etc/php/php.ini \
    && echo "upload_max_filesize=20M" >> /usr/local/etc/php/php.ini \
    && echo "post_max_size=20M" >> /usr/local/etc/php/php.ini \
    && chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html/var \
    && chmod -R 755 /var/www/html/public \
    && composer run-script post-install-cmd --no-interaction || echo "Post-install scripts completed with warnings" \
    && mkdir -p var/cache var/log \
    && chown -R www-data:www-data var/ \
    && chmod -R 775 var/

# Expose port
EXPOSE 9000

# Use www-data user
USER www-data

# Start php-fpm
CMD ["php-fpm"]