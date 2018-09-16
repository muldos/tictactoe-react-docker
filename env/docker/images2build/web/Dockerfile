FROM php:7.1-apache

MAINTAINER David ROBIN <david.robin@gpsante.fr>

# Pour éviter les décalage horaire dans les containers
ENV TZ Europe/Paris
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
	&& echo $TZ > /etc/timezone \
	&& dpkg-reconfigure -f noninteractive tzdata

# Modify UID of www-data into UID of local user and GID of group www-data
ARG WWW_UID=33
RUN usermod -u ${WWW_UID} www-data && groupmod -g ${WWW_UID} www-data
# PHP extensions et lib systeme requises
RUN set -ex; \
	\
	if command -v a2enmod; then \
		a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		zlib1g-dev \
		libicu57 \
		libjpeg62-turbo-dev \
		libfreetype6-dev \
		libmcrypt4 \
		libmcrypt-dev \		
		libpng-dev \
		libpq-dev \
		libicu-dev \		
	; \
	\
	docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr; \
	docker-php-ext-install -j "$(nproc)" \
		gd \
		intl \
		exif \
		fileinfo \
		intl \
		opcache \
		pdo \
		pdo_mysql \
		mcrypt \
		zip \
	; \
	pecl install redis-3.1.6 \
    && pecl install xdebug-2.5.5 \
    && pecl install mongodb-1.4.2 \    
    && docker-php-ext-enable redis xdebug mongodb \
    ; \
# cleaning des build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*
COPY conf/cms.ini /usr/local/etc/php/conf.d/cms.ini
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini
# Installation de composer
RUN php -r 'readfile("https://getcomposer.org/installer");' > composer-setup.php \
	&& php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
	&& rm -f composer-setup.php \
	&& chown -R www-data /var/www \
	&& chgrp -R www-data /var/www