FROM php:8.1-fpm

# Set environment variables
ARG TZ=UTC
ENV TZ ${TZ}

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

# Install dependencies
RUN apt-get update && apt-get install -y \
    mariadb-client \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    libcurl4-nss-dev \
    libc-client-dev \
    libkrb5-dev \
    firebird-dev \
    libicu-dev \
    libxml2-dev \
    libxslt1-dev \
    autoconf \
    wget \
    zip \
    unzip \
    cron \
    git \
    libzip-dev \
    locales-all \
    libonig-dev \
    wkhtmltopdf

RUN install-php-extensions \
    bcmath \
    exif \
    gd \
    imagick \
    imap \
    intl \
    mysqli \
    pdo_mysql \
    redis \
    soap \
    xdebug \
    xsl \
    zip

# Set the timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Comment out xdebug extension line per default
RUN sed -i 's/^zend_extension=/;zend_extension=/g' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

# Copy the php-fpm config
COPY ./php.ini /usr/local/etc/php/conf.d/
COPY ./fpm.conf /usr/local/etc/php-fpm.d/

USER root

# Configure non-root user.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

# Set the proper permissions
RUN groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data

ENV IP_SOURCE="https://github.com/InvoicePlane/InvoicePlane/releases/download" \
    IP_VERSION="v1.6.0" \
    MYSQL_HOST="mariadb_10_4" \
    MYSQL_USER="ipdevdb" \
    MYSQL_PASSWORD="ipdevdb" \
    MYSQL_DB="invoiceplane_db" \
    MYSQL_PORT="3306" \
    IP_URL="http://ivplv1.local" \
    HOST_URL="ivplv1.local" \
    DISABLE_SETUP="false"

RUN mkdir -p /config
COPY ./start.sh /config
# copy invoiceplane sources to web dir
ADD ${IP_SOURCE}/v1.6.0/v1.6.0.zip /tmp/
RUN cd /tmp && \
    unzip /tmp/v1.6.0.zip && \
    cp -r ip/* /var/www/html/ && \
    chmod +x /config/start.sh;

#
#--------------------------------------------------------------------------
# Final Touches
#--------------------------------------------------------------------------
#

# Cleanup all downloaded packages
RUN apt-get -y autoclean && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm /var/log/lastlog /var/log/faillog

RUN chown www-data:www-data /var/www/html/* -R;

USER www-data

WORKDIR /var/www/html

VOLUME /var/www/html/uploads
EXPOSE 80
ENTRYPOINT ["/config/start.sh"]
CMD ["php-fpm"]