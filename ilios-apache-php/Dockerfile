# start with the official Composer image and name it
#FROM composer:1.5.1 AS composer

FROM php:7.1.13-apache AS apache-php

MAINTAINER Ilios Project Team <support@iliosproject.org>

# copy the Composer PHAR from the Composer image into the apache-php image
#COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN apt-get update \
    && apt-get install -y \
    && apt-get install libldap2-dev -y \
    && apt-get install zlib1g-dev \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
    && docker-php-ext-install ldap \
    && docker-php-ext-install zip \
    && rm -rf /var/lib/apt/lists/* \
    && pecl channel-update pecl.php.net \
    && pecl install apcu \
    && docker-php-ext-enable apcu

VOLUME /data

# get via the known working method
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

ENV COMPOSER_HOME=/tmp \
  SYMFONY_ENV=prod \
  ILIOS_API_ENVIRONMENT=prod \
  ILIOS_API_DEBUG=false \
  ILIOS_DATABASE_DRIVER=pdo_mysql \
  ILIOS_DATABASE_HOST=db \
  ILIOS_DATABASE_PORT=~ \
  ILIOS_DATABASE_NAME=ilios \
  ILIOS_DATABASE_USER=ilios \
  ILIOS_DATABASE_PASSWORD=ilios \
  ILIOS_DATABASE_MYSQL_VERSION=5.7 \
  ILIOS_MAILER_TRANSPORT=smtp \
  ILIOS_MAILER_HOST=127.0.0.1 \
  ILIOS_MAILER_USER=~ \
  ILIOS_MAILER_PASSWORD=~ \
  ILIOS_LOCALE=en \
  ILIOS_SECRET=ThisTokenIsNotSoSecretChangeIt \
  ILIOS_AUTHENTICATION_TYPE=form \
  ILIOS_LEGACY_PASSWORD_SALT=null \
  ILIOS_FILE_SYSTEM_STORAGE_PATH=/data \
  ILIOS_INSTITUTION_DOMAIN=example.com \
  ILIOS_SUPPORTING_LINK=null \
  ILIOS_LDAP_AUTHENTICATION_HOST=null \
  ILIOS_LDAP_AUTHENTICATION_PORT=null \
  ILIOS_LDAP_AUTHENTICATION_BIND_TEMPLATE=null \
  ILIOS_LDAP_DIRECTORY_URL=null \
  ILIOS_LDAP_DIRECTORY_USER=null \
  ILIOS_LDAP_DIRECTORY_PASSWORD=null \
  ILIOS_LDAP_DIRECTORY_SEARCH_BASE=null \
  ILIOS_LDAP_DIRECTORY_CAMPUS_ID_PROPERTY=null \
  ILIOS_LDAP_DIRECTORY_USERNAME_PROPERTY=null \
  ILIOS_SHIBBOLETH_AUTHENTICATION_LOGIN_PATH=null \
  ILIOS_SHIBBOLETH_AUTHENTICATION_LOGOUT_PATH=null \
  ILIOS_SHIBBOLETH_AUTHENTICATION_USER_ID_ATTRIBUTE=null \
  ILIOS_TIMEZONE='America/Los_Angeles' \
  ILIOS_REQUIRE_SECURE_CONNECTION=false \
  ILIOS_KEEP_FRONTEND_UPDATED=true \
  ILIOS_FRONTEND_RELEASE_VERSION=null \
  ILIOS_CAS_AUTHENTICATION_SERVER=null \
  ILIOS_CAS_AUTHENTICATION_VERSION=3 \
  ILIOS_CAS_AUTHENTICATION_VERIFY_SSL=true \
  ILIOS_CAS_AUTHENTICATION_CERTIFICATE_PATH=null \
  ILIOS_ENABLE_TRACKING=false \
  ILIOS_TRACKING_CODE=UA-XXXXXXXX-1 \
  COMPOSER_ALLOW_SUPERUSER=1 

COPY ["src/ilios", "/var/www/ilios"]

# make sure we're not attempting to building with ANY caching
RUN rm -Rf /var/www/ilios/var/cache
RUN rm -Rf /var/www/ilios/vendor

RUN mkdir -p \
    /var/www/ilios/var \
    /var/www/ilios/var/cache \
    /var/www/ilios/var/logs \
    /var/www/ilios/var/session \
    /var/www/ilios/var/tmp \
    /var/www/ilios/vendor

RUN chown -R www-data:www-data /var/www/ilios

RUN sed -i'' 's%/var/www/html%/var/www/ilios/web%g' /etc/apache2/sites-enabled/000-default.conf

RUN chsh -s /bin/bash www-data

USER www-data
RUN /usr/bin/composer install \
  --working-dir /var/www/ilios \
  --prefer-dist \
  --no-dev \
  --no-progress \
  --no-interaction \
  --no-suggest \
  --classmap-authoritative

RUN /var/www/ilios/bin/console ilios:maintenance:update-frontend

USER root
RUN chsh -s /usr/sbin/nologin www-data

EXPOSE 80:80

