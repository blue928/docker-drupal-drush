FROM php:7.4-apache-buster

# Install packages
RUN apt-get update && apt-get install --no-install-recommends -y \
	curl \
	wget \
	vim \
	git \
	unzip \
	mariadb-client

RUN if command -v a2enmod; then \
	a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
	libfreetype6-dev \
	libjpeg-dev \
	libpng-dev \
	libpq-dev \
	libzip-dev \
	; \
	\
	docker-php-ext-configure gd \
	--with-freetype \
	--with-jpeg=/usr \
	; \
	\
	docker-php-ext-install -j "$(nproc)" \
	gd \
	opcache \
	pdo_mysql \
	pdo_pgsql \
	zip \
	; 

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer && \
    ln -s /root/.composer/vendor/bin/drush /usr/local/bin/drush


# Install PHP Coder Sniffer
# https://www.drupal.org/node/1419988
RUN composer global require drupal/coder; \
    export PATH="$PATH:$HOME/.composer/vendor/bin"; \
	composer global require drupal/coder dealerdirect/phpcodesniffer-composer-installer

# Install Drush
RUN composer global require drush/drush:8.3.3 --with-all-dependencies && \
    composer global update

# Don't install drush with composer. See this SO issue
# https://stackoverflow.com/questions/67788318/why-despite-drush-being-installed-via-composer-global-install-during-image-b
#RUN wget https://github.com/drush-ops/drush/releases/download/8.4.5/drush.phar; \
#	mv drush.phar /usr/local/bin/drush; \
#	chmod +x /usr/local/bin/drush

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
	echo 'opcache.memory_consumption=128'; \
	echo 'opcache.interned_strings_buffer=8'; \
	echo 'opcache.max_accelerated_files=4000'; \
	echo 'opcache.revalidate_freq=60'; \
	echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# https://www.drupal.org/node/3060/release
ENV DRUPAL_VERSION 7.80
ENV DRUPAL_MD5 cfbbc4cde2782f4e7298db3991ca6f56

RUN curl -fSL "https://ftp.drupal.org/files/projects/drupal-${DRUPAL_VERSION}.tar.gz" -o drupal.tar.gz; \
	echo "${DRUPAL_MD5} *drupal.tar.gz" | md5sum -c -; \
	tar -xz --strip-components=1 -f drupal.tar.gz; \
	rm drupal.tar.gz; \
	chown -R www-data:www-data sites modules themes


# Clean repository
RUN apt-get clean && rm -rf /var/lib/apt/lists/*