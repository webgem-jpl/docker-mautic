FROM php:7.0-apache
MAINTAINER Michael Babker <michael.babker@mautic.org> (@mbabker)

# Install PHP extensions
RUN apt-get update && apt-get install --no-install-recommends -y \
    cron \
    libc-client-dev \
    libicu-dev \
    libkrb5-dev \
    libmcrypt-dev \
    libssl-dev \
    unzip \
    zip \
    && rm /etc/cron.daily/*

RUN docker-php-ext-configure imap --with-imap --with-imap-ssl --with-kerberos \
    && docker-php-ext-install imap intl mbstring mcrypt mysqli pdo pdo_mysql zip

# Define Mautic version and expected SHA1 signature
ENV MAUTIC_VERSION 2.12.1
ENV MAUTIC_SHA1 68eae0b27442b32f24d41cb4bea8fbfaa2f9772e

ENV MAUTIC_RUN_CRON_JOBS true
ENV MAUTIC_DB_USER root
ENV MAUTIC_DB_NAME mautic

# Opcache Config
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Download package
RUN curl -o mautic.zip -SL https://s3.amazonaws.com/mautic/releases/${MAUTIC_VERSION}.zip \
	&& echo "$MAUTIC_SHA1 *mautic.zip" | sha1sum -c - \
	&& unzip mautic.zip -d /usr/src/mautic \
	&& rm mautic.zip \
  && tar cf - --one-file-system -C /usr/src/mautic . | tar xf - \
  && chown -R www-data:www-data /var/www/html


#Mautic Cache Files
# Copy init scripts and custom .htaccess
COPY docker-entrypoint.sh /entrypoint.sh
COPY makeconfig.php /makeconfig.php
COPY makedb.php /makedb.php
COPY mautic.crontab /etc/cron.d/mautic
COPY mautic-php.ini /usr/local/etc/php/conf.d/mautic-php.ini

#Put all Mautic instance config files into one directory
RUN cd /var/www/html/ && \
    mkdir -p config \
    app/local/cache/prod \
    app/local/config \
    app/local/themes \
    app/local/idp \
    app/media/files \
    app/media/images \
    app/local/plugins

COPY paths_local.php /var/www/html/app/config/
COPY parameters_local.php /var/www/html/app/config/

RUN  chown -R www-data:www-data /var/www/html

RUN mkdir -p /var/log/mautic && \
    chown -R www-data:www-data /var/log/mautic

# Enable Apache Rewrite Module
RUN a2enmod rewrite

ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
