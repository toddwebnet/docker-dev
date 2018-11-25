FROM php:7.2-fpm
MAINTAINER James Todd <toddwebnet@gmail.com>

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    curl \
    git \
    libmemcached-dev \
    libz-dev \
    libpq-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libssl-dev \
    libmcrypt-dev \
    supervisor
  && rm -rf /var/lib/apt/lists/*


RUN docker-php-ext-install pdo_mysql \
&& docker-php-ext-install pdo_pgsql \
&& docker-php-ext-configure gd \
  --with-jpeg-dir=/usr/lib \
  --with-freetype-dir=/usr/include/freetype2 && \
  docker-php-ext-install gd

RUN apt-get update -yqq && \
  apt-get install -y apt-utils && \
  pecl channel-update pecl.php.net


###########################################################################
# pgsql
###########################################################################

ARG INSTALL_PGSQL=false

RUN if [ ${INSTALL_PGSQL} = true ]; then \
    # Install the pgsql extension
    docker-php-ext-install pgsql \
;fi


###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    # Install Php Redis Extension
    printf "\n" | pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis \
;fi


###########################################################################
# MongoDB:
###########################################################################

ARG INSTALL_MONGO=false

RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install mongo && \
      docker-php-ext-enable mongo \
    ;fi && \
    pecl install mongodb && \
    docker-php-ext-enable mongodb \
;fi


###########################################################################
# ZipArchive:
###########################################################################

ARG INSTALL_ZIP_ARCHIVE=false

RUN if [ ${INSTALL_ZIP_ARCHIVE} = true ]; then \
    apt-get install libzip-dev -y && \
    docker-php-ext-configure zip --with-libzip && \
    # Install the zip extension
    docker-php-ext-install zip \
;fi


###########################################################################
# PHP Memcached:
###########################################################################

ARG INSTALL_MEMCACHED=false

RUN if [ ${INSTALL_MEMCACHED} = true ]; then \
    # Install the php memcached extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      curl -L -o /tmp/memcached.tar.gz "https://github.com/php-memcached-dev/php-memcached/archive/2.2.0.tar.gz"; \
    else \
      curl -L -o /tmp/memcached.tar.gz "https://github.com/php-memcached-dev/php-memcached/archive/php7.tar.gz"; \
    fi \
    && mkdir -p memcached \
    && tar -C memcached -zxvf /tmp/memcached.tar.gz --strip 1 \
    && ( \
        cd memcached \
        && phpize \
        && ./configure \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r memcached \
    && rm /tmp/memcached.tar.gz \
    && docker-php-ext-enable memcached \
;fi


###########################################################################
# Mysqli Modifications:
###########################################################################

ARG INSTALL_MYSQLI=false

RUN if [ ${INSTALL_MYSQLI} = true ]; then \
    docker-php-ext-install mysqli \
;fi


###########################################################################
# GHOSTSCRIPT:
###########################################################################

ARG INSTALL_GHOSTSCRIPT=false

RUN if [ ${INSTALL_GHOSTSCRIPT} = true ]; then \
    # Install the ghostscript extension
    # for PDF editing
    apt-get install -y \
    poppler-utils \
    ghostscript \
;fi

###########################################################################
# LDAP:
###########################################################################

ARG INSTALL_LDAP=false

RUN if [ ${INSTALL_LDAP} = true ]; then \
    apt-get install -y libldap2-dev && \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
    docker-php-ext-install ldap \
;fi

###########################################################################
# SQL SERVER:
###########################################################################

ARG INSTALL_MSSQL=false

RUN set -eux; if [ ${INSTALL_MSSQL} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      apt-get -y install freetds-dev libsybdb5 \
      && ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/libsybdb.so \
      && docker-php-ext-install mssql pdo_dblib \
      && php -m | grep -q 'mssql' \
      && php -m | grep -q 'pdo_dblib' \
    ;else \
      ###########################################################################
      # Ref from https://github.com/Microsoft/msphpsql/wiki/Dockerfile-for-adding-pdo_sqlsrv-and-sqlsrv-to-official-php-image
      ###########################################################################
      # Add Microsoft repo for Microsoft ODBC Driver 13 for Linux
      apt-get install -y apt-transport-https gnupg \
      && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
      && curl https://packages.microsoft.com/config/debian/9/prod.list > /etc/apt/sources.list.d/mssql-release.list \
      && apt-get update -yqq \
      # Install Dependencies
      && ACCEPT_EULA=Y apt-get install -y unixodbc unixodbc-dev libgss3 odbcinst msodbcsql17 locales \
      && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
      # link local aliases
      && ln -sfn /etc/locale.alias /usr/share/locale/locale.alias \
      && locale-gen \
      # Install pdo_sqlsrv and sqlsrv from PECL. Replace pdo_sqlsrv-4.1.8preview with preferred version.
      && pecl install pdo_sqlsrv sqlsrv \
      && docker-php-ext-enable pdo_sqlsrv sqlsrv \
      && php -m | grep -q 'pdo_sqlsrv' \
      && php -m | grep -q 'sqlsrv' \
    ;fi \
;fi

###########################################################################
# Image optimizers:
###########################################################################

USER root

ARG INSTALL_IMAGE_OPTIMIZERS=false

RUN if [ ${INSTALL_IMAGE_OPTIMIZERS} = true ]; then \
    apt-get install -y jpegoptim optipng pngquant gifsicle \
;fi

###########################################################################
# ImageMagick:
###########################################################################

USER root

ARG INSTALL_IMAGEMAGICK=false

RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -y libmagickwand-dev imagemagick && \
    pecl install imagick && \
    docker-php-ext-enable imagick \
;fi

###########################################################################
# IMAP:
###########################################################################

ARG INSTALL_IMAP=false

RUN if [ ${INSTALL_IMAP} = true ]; then \
    apt-get install -y libc-client-dev libkrb5-dev && \
    rm -r /var/lib/apt/lists/* && \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
    docker-php-ext-install imap \
;fi



###########################################################################
# Calendar:
###########################################################################

USER root

ARG INSTALL_CALENDAR=false

RUN if [ ${INSTALL_CALENDAR} = true ]; then \
    docker-php-ext-configure calendar && \
    docker-php-ext-install calendar \
;fi



RUN rm /var/cache/apk/* \
    && mkdir -p /var/www

#
#--------------------------------------------------------------------------
# Optional Supervisord Configuration
#--------------------------------------------------------------------------
#
# Modify the ./supervisor.conf file to match your App's requirements.
# Make sure you rebuild your container with every change.
#

COPY supervisord.conf /etc/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisord.conf"]

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# If you need to modify this image, feel free to do it right here.
#
    # -- Your awesome modifications go here -- #


###########################################################################
# Check PHP version:
###########################################################################

RUN php -v | head -n 1 | grep -q "PHP 7.2."



# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog
#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

WORKDIR /etc/supervisor/conf.d/
