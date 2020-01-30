FROM composer:1.8.5 as composer
WORKDIR /var/www
# Copy composer files from project root into composer container's working dir
COPY ./composer.json /var/www/composer.json

# Run composer to build dependencies in vendor folder
RUN set -xe \
 && composer install --no-dev --no-scripts --no-suggest --no-interaction --prefer-dist --optimize-autoloader

# Copy everything from project root into composer container's working dir
COPY . /var/www

# Generated optimized autoload files containing all classes from vendor folder and project itself
RUN composer dump-autoload --no-dev --optimize --classmap-authoritative

FROM php:7.2-apache
MAINTAINER Marco Aurélio <malf88@gmail.com>

WORKDIR /var/www

COPY ./ /var/www/


ENV APACHE_DOCUMENT_ROOT /var/www/


# Install apache, PHP, and supplimentary programs. openssh-server, curl, and lynx-cur are for debugging the container.
RUN apt-get update && apt-get -y upgrade && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        nano \
        libcurl4-gnutls-dev \
        libxml2-dev \
        libpng-dev \
        libpq-dev \
        git && \
    docker-php-ext-install \
        bcmath \
        calendar \
        curl \
        dom \
        fileinfo \
        gd \
        hash \
        json \
        mbstring \
        mysqli \
        pdo \
        pdo_mysql \
        sockets \
        zip && \
    pecl install xdebug && \
    docker-php-ext-enable xdebug && \
    curl -sL https://deb.nodesource.com/setup_10.x | bash - \
    && apt-get update && apt-get install -y nodejs && \
    php -r "copy('http://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');" && \
    mv composer.phar /usr/local/bin/composer

#RUN apt-get install -y nodejs
# Enable apache mods.
RUN a2enmod rewrite

RUN mv $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini

RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" $PHP_INI_DIR/php.ini
RUN sed -i "s/display_errors = .*$/display_errors = On/" $PHP_INI_DIR/php.ini
RUN echo "ServerName localhost:80" >> /etc/apache2/apache2.conf

# Manually set up the apache environment variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/www/storage/logs/
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV COMPOSER_ALLOW_SUPERUSER 1
COPY --from=composer /var/www/vendor /var/www/vendor
# Expose apache.
EXPOSE 80
COPY ./docker/cron.sh /usr/local/bin/start

RUN chmod u+x /usr/local/bin/start

ADD ./docker/000-default.conf /etc/apache2/sites-enabled/000-default.conf

CMD ["/usr/local/bin/start"]
# By default start up apache in the foreground, override with /bin/bash for interative.
