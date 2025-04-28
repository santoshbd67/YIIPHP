FROM php:8.1-apache

# Install system packages
RUN apt-get update && apt-get install -y \
    unzip \
    git \
    libzip-dev \
    && docker-php-ext-install pdo pdo_mysql zip

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Copy app files
COPY ./ /var/www/html/

# Install PHP dependencies
WORKDIR /var/www/html
RUN composer install --no-interaction --prefer-dist --optimize-autoloader

# Set permissions
RUN mkdir -p /var/www/html/runtime /var/www/html/web/assets \
    && chmod -R 777 /var/www/html/runtime /var/www/html/web/assets

# Apache config
RUN a2enmod rewrite

# Update DocumentRoot
RUN sed -i 's|DocumentRoot /var/www/html|DocumentRoot /var/www/html/web|g' /etc/apache2/sites-available/000-default.conf

# Correct webdir.conf
RUN printf "<Directory /var/www/html/web>\n\
    AllowOverride All\n\
    Require all granted\n\
</Directory>\n" > /etc/apache2/conf-available/webdir.conf \
    && a2enconf webdir

EXPOSE 80

CMD ["apache2-foreground"]

