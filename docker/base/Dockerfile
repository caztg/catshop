FROM php:7.1-apache

RUN a2enmod rewrite

# Memory Limit
RUN echo "memory_limit=256M" > $PHP_INI_DIR/conf.d/memory-limit.ini

# Time Zone
RUN echo "date.timezone=Asia/Hong_Kong" > $PHP_INI_DIR/conf.d/date_timezone.ini

# Packages
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
  libfreetype6-dev \
  libjpeg62-turbo-dev \
  libmcrypt-dev \
  libpng-dev \
  libbz2-dev \
  php-pear \
  curl \
  git \
  unzip \
  mysql-client \
  && docker-php-ext-install -j$(nproc) zip bz2 bcmath pdo_mysql mysqli mbstring opcache \
  && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install -j$(nproc) gd

# Environmental Variables
ENV COMPOSER_HOME /root/composer

# Add global binary directory to path
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    composer config -g github-oauth.github.com b920a503e5bb62ee4f4bcfeb1dfc76bad0706a2c

# Set up the application directory
WORKDIR /app
ENV APACHE_DOCUMENT_ROOT /app/web
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf && \
    sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# Install Drupal Commerce 2.x via drupalcommerce/project-base
RUN composer create-project drupalcommerce/project-base \
 --stability dev \
 --no-interaction \
 .

RUN composer require "drupal/core:8.4.3" \
                    "drupal/facets:1.x-dev" \
                    "drupal/devel:1.x-dev" \
                    "drupal/default_content" \
                    "drupal/file_entity:2.x-dev" \
                    "drupal/restui" \
                    "drupal/openapi"


RUN curl -L -o drush.phar https://github.com/drush-ops/drush-launcher/releases/download/0.4.2/drush.phar && \
    chmod +x drush.phar && \
    mv drush.phar /usr/local/bin/drush && \
    drush self-update

RUN curl https://drupalconsole.com/installer -L -o drupal.phar && \
    mv drupal.phar /usr/local/bin/drupal && \
    chmod +x /usr/local/bin/drupal && \
    drupal self-update
